Before a user enters a room, or during a call, TRTC will check the user's network quality in advance. If the quality is low, you should recommend the user to change the network environment to ensure smooth calls.

This document describes how to check the network quality before a call based on the [NETWORK_QUALITY](https://web.sdk.qcloud.com/trtc/webrtc/doc/en/module-ClientEvent.html#.NETWORK_QUALITY) event.

## Implementation Process

1. Call [TRTC.createClient](https://web.sdk.qcloud.com/trtc/webrtc/doc/en/TRTC.html#.createClient) to create two clients: `uplinkClient` and `downlinkClient`.
2. Make the two clients enter the same room.
3. Use `uplinkClient` to push a stream. Listen on the [NETWORK_QUALITY](https://web.sdk.qcloud.com/trtc/webrtc/doc/en/module-ClientEvent.html#.NETWORK_QUALITY) event to check the upstream network quality.
4. Use `downlinkClient` to pull a stream. Listen on the [NETWORK_QUALITY](https://web.sdk.qcloud.com/trtc/webrtc/doc/en/module-ClientEvent.html#.NETWORK_QUALITY) event to check the downstream network quality.
5. The whole process lasts for about 15 seconds, and the average network quality is taken eventually to evaluate the upstream and downstream network quality.

> The check process may incur a small amount of [basic service fee](https://cloud.tencent.com/document/product/647/17157#.E5.9F.BA.E7.A1.80.E6.9C.8D.E5.8A.A1). If the push resolution is not specified, the stream will be pushed at a resolution of 640x480 by default.

## API Call Sequence

![network-quality-call-sequence](./assets/network-quality-call-sequence.png)

## Sample Code

```js
let uplinkClient = null; // Check the upstream network quality
let downlinkClient = null; // Check the downstream network quality
let localStream = null; // Stream for test
let testResult = {
  // Record the upstream network quality data
  uplinkNetworkQualities: [],
  // Record the downstream network quality data
  downlinkNetworkQualities: [],
  average: {
    uplinkNetworkQuality: 0,
    downlinkNetworkQuality: 0
  }
}

// 1. Check the upstream network quality
async function testUplinkNetworkQuality() {
  uplinkClient = TRTC.createClient({
    sdkAppId: 0, // Enter `sdkAppId`
    userId: 'user_uplink_test',
    userSig: '', // `userSig` of `uplink_test`
    mode: 'rtc'
  });

  localStream = TRTC.createStream({ audio: true, video: true });
  // Set the video profile based on the actual business scenario
  localStream.setVideoProfile('480p'); 
  await localStream.initialize();

  uplinkClient.on('network-quality', event => {
    const { uplinkNetworkQuality } = event;
    testResult.uplinkNetworkQualities.push(uplinkNetworkQuality);
  });

  // Enter the room for test. The room ID must be random to avoid conflicts
  await uplinkClient.join({ roomId: 8080 }); 
  await uplinkClient.publish(localStream);
}

// 2. Check the downstream network quality
async function testDownlinkNetworkQuality() {
  downlinkClient = TRTC.createClient({
    sdkAppId: 0, // Enter `sdkAppId`
    userId: 'user_downlink_test',
    userSig: '', // userSig
    mode: 'rtc'
  });

  downlinkClient.on('stream-added', async event => {
    await downlinkClient.subscribe(event.stream, { audio: true, video: true });
		// Start listening on the network quality event after successful subscription
    downlinkClient.on('network-quality', event => {
      const { downlinkNetworkQuality } = event;
      testResult.downlinkNetworkQualities.push(downlinkNetworkQuality);
    });
  })
  // Enter the room for test. The room ID must be random to avoid conflicts
  await downlinkClient.join({ roomId: 8080 });
}

// 3. Start the check
testUplinkNetworkQuality();
testDownlinkNetworkQuality();

// 4. Stop the check after 15 seconds and calculate the average network quality
setTimeout(() => {
  // Calculate the average upstream network quality
  if (testResult.uplinkNetworkQualities.length > 0) {
    testResult.average.uplinkNetworkQuality = Math.ceil(
      testResult.uplinkNetworkQualities.reduce((value, current) => value + current, 0) / testResult.uplinkNetworkQualities.length
    );
  }

  if (testResult.downlinkNetworkQualities.length > 0) {
    // Calculate the average downstream network quality
    testResult.average.downlinkNetworkQuality = Math.ceil(
      testResult.downlinkNetworkQualities.reduce((value, current) => value + current, 0) / testResult.downlinkNetworkQualities.length
    );
  }
    
  // The check ends. Clear the relevant status.
  uplinkClient.leave();
  downlinkClient.leave();
  localStream.close();
}, 15 * 1000);
```

## Result Analysis

After performing the above steps, you can get the average upstream and downstream network quality. The enumerated values of the network quality are as listed below:

| Value | Description                                                         |
| :--- | :----------------------------------------------------------- |
| 0    | Unknown network quality, which indicates that the current client instance hasn't established an upstream/downstream connection    |
| 1    | Excellent network quality                                                 |
| 2    | Good network quality                                                 |
| 3    | Average network quality                                                 |
| 4    | Poor network quality                                                   |
| 5    | Awful network quality                                                 |
| 6    | The network connection has been closed. Note: if the downstream network quality is this value, it indicates that all downstream connections have been closed |

> Suggestion: if the network quality is above 3, you should prompt the user to check the network and try changing the network environment; otherwise, normal audio/video calls cannot be guaranteed. <br>You can also use the following methods to reduce the bandwidth consumption:
> - If the upstream network quality is above 3, you can call the [LocalStream.setVideoProfile()](https://web.sdk.qcloud.com/trtc/webrtc/doc/en/LocalStream.html#setVideoProfile) API to reduce the bitrate or call [LocalStream.muteVideo()](https://web.sdk.qcloud.com/trtc/webrtc/doc/en/LocalStream.html#muteVideo) to disable the video so as to reduce the upstream bandwidth consumption.
> - If the downstream network quality is above 3, you can subscribe to only the substream as instructed in [Enabling Primary Stream/Substream Transfer](https://web.sdk.qcloud.com/trtc/webrtc/doc/en/tutorial-27-advanced-small-stream.html) or audio to reduce the downstream bandwidth consumption.

