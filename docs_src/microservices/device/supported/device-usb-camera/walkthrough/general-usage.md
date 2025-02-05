# General Usage 
This document will describe how to execute some of the most important types of commands used with the device service.

## Start Video Streaming
Unless the device service is configured to stream video from the camera automatically, a `StartStreaming` command must be sent to the device service.

There are two types of options:
- The options that start with `Input` as a prefix are used for camera configuration, such as specifying the image size and pixel format.
- The options that start with `Output` as a prefix are used for video output configuration, such as specifying aspect ratio and quality.

These options can be passed in through Object value when calling StartStreaming.

Query parameter:  
- `device name`: The name of the camera

!!! example - "Example StartStreaming Command"
    ```shell
    curl -X PUT -d '{
        "StartStreaming": {
        "InputImageSize": "640x480",
        "OutputVideoQuality": "5"
        }
    }' http://localhost:59882/api/v3/device/name/<device name>/StartStreaming
    ```

Supported Input options:  

- `InputFps`: Ignore original timestamps and instead generate timestamps assuming constant frame rate fps. (default - same as source)  
- `InputImageSize`: Specifies the image size of the camera. The format is `wxh`, for example "640x480". (default - automatically selected by FFmpeg)  
- `InputPixelFormat`: Set the preferred pixel format (for raw video). (default - automatically selected by FFmpeg)  

Supported Output options: 

- `OutputFrames`: Set the number of video frames to output. (default - no limitation on frames)  
- `OutputFps`: Duplicate or drop input frames to achieve constant output frame rate fps. (default - same as InputFps)  
- `OutputImageSize`: Performs image rescaling. The format is `wxh`, for example "640x480". (default - same as InputImageSize)  
- `OutputAspect`: Set the video display aspect ratio specified by aspect. For example "4:3", "16:9". (default - same as source)  
- `OutputVideoCodec`: Set the video codec. For example "mpeg4", "h264". (default - mpeg4)  
- `OutputVideoQuality`: Use fixed video quality level. Range is a integer number between 1 to 31, with 31 being the worst quality. (default - dynamically set by FFmpeg)  


### Determine Stream Uri of Camera
The device service provides a way to determine the stream URI of a camera.

Query parameter:  
- `device name`: The name of the camera
!!! example - "Example StreamURI Command"
    ```bash
    curl -s http://localhost:59882/api/v3/device/name/<device name>/StreamURI | jq -r '"StreamURI: " + '.event.readings[].value''
    ```

The response to the above call should look similar to the following:
```
StreamURI: rtsp://localhost:8554/stream/NexiGo_N930AF_FHD_Webcam__NexiG-20201217010
```

### Play the RTSP stream. 

mplayer can be used to stream. The command follows this format: 

```bash
mplayer rtsp://'<username>:<password>'@<IP address>:<port>/<streamname>`.
```

Using the `streamURI` returned from the previous step, run mplayer:

!!! example - "Example Stream Command"
    ```bash
    mplayer rtsp://'admin:pass'@localhost:8554/stream/NexiGo_N930AF_FHD_Webcam__NexiG-20201217010
    ```


To shut down mplayer, use the ctrl-c command.


### Stop Video Streaming
To stop the usb camera from live streaming, use the following command:

Query parameter:  
- `device name`: The name of the camera

!!! example - "Example StopStreaming Command"
    ```shell
    curl -X PUT -d '{
        "StopStreaming": "true"
    }' http://localhost:59882/api/v3/device/name/<device name>/StopStreaming
    ```

## Optional: Shutting Down

To stop all EdgeX services (containers), execute the `make down` command:

1. Navigate to the `edgex-compose/compose-builder` directory.

    ```shell
    cd ~/edgex/edgex-compose/compose-builder
    ```

1. Run this command
    ```bash
    make down
    ```

1. To shut down and delete all volumes, run this command

    !!! Warning 
        This will delete all edgex-related data.  

    ```bash
    make clean
    ```

## Troubleshooting
### StreamingStatus
To verify the usb camera is set to stream video, use the command below. 

Query parameter:  
- `device name`: The name of the camera

```bash
curl http://localhost:59882/api/v3/device/name/<device name>/StreamingStatus | jq -r '"StreamingStatus: " + (.event.readings[].objectValue.IsStreaming|tostring)'
```

If the StreamingStatus is false, the camera is not configured to stream video. Please try the Start Video Streaming section again [here](#start-video-streaming).

### V4L2 error
If you get an error like this:
```
.../go4vl@v0.0.2/v4l2/capability.go:48:33: could not determine kind of name for C.V4L2_CAP_IO_MC
.../go4vl@v0.0.2/v4l2/capability.go:46:33: could not determine kind of name for C.V4L2_CAP_META_OUTPUT
```

You are missing the appropriate kernel headers needed by the `github.com/vladimirvivien/go4vl` module.
One possible solution is to manually download and install a more recent version of the libc-dev for your OS.

In the case of Ubuntu 20.04, one is not available in the normal repositories, so you can get it via these steps:

```
wget https://launchpad.net/~canonical-kernel-team/+archive/ubuntu/bootstrap/+build/20950478/+files/linux-libc-dev_5.10.0-14.15_amd64.deb
sudo dpkg -i linux-libc-dev_5.10.0-14.15_amd64.deb
```    
