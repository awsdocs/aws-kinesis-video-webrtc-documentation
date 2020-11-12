# Client Metrics for the WebRTC C SDK<a name="kvswebrtc-reference"></a>

Applications built with Amazon Kinesis Video Streams with WebRTC are comprised of various moving parts, including networking, signaling, candidates exchange, peer connection, and data exchange\. Kinesis Video Streams with WebRTC in C supports various client\-side metrics that enable you to monitor and track the performance and usage of these components in your applications\. The supported metrics fall into two major categories: custom metrics defined specifically for the Kinesis Video Streams' implementation of signaling and networking, and media and data\-related protocol\-specific metrics that are derived from the [W3C](https://www.w3.org/TR/webrtc-stats/) standard\. Note that only a subset of the W3C standard metrics is currently supported for Kinesis Video Streams with WebRTC in C\. 

**Topics**
+ [Signaling Metrics](#kvswebrtc-reference-signaling)
+ [W3C Standard Metrics Supported for WebRTC C SDK](#kvswebrtc-reference-w3cstandard)

## Signaling Metrics<a name="kvswebrtc-reference-signaling"></a>

Signaling metrics can be used to understand how the signaling client behaves while your application is running\. You can use the `STATUS signalingClientGetMetrics (SIGNALING_CLIENT_HANDLE, PSignalingClientMetrics)` API to obtain these signaling metrics\. Here's an example usage pattern:

```
SIGNALING_CLIENT_HANDLE signalingClientHandle;
SignalingClientMetrics signalingClientMetrics;
STATUS retStatus = signalingClientGetMetrics(signalingClientHandle, &signalingClientMetrics);
printf("Signaling client connection duration: %" PRIu64 " ms",
       (signalingClientMetrics.signalingClientStats.connectionDuration / HUNDREDS_OF_NANOS_IN_A_MILLISECOND));
```

The Definition of `signalingClientStats` can be found in [Stats\.h](https://github.com/awslabs/amazon-kinesis-video-streams-webrtc-sdk-c/blob/master/src/include/com/amazonaws/kinesis/video/webrtcclient/Stats.h)\.

The following signaling metrics are currently supported:


****  

| Metric | Description | 
| --- | --- | 
| cpApiCallLatency | Calculate latency for control plane API calls\. Calculation is done using Exponential Moving Average \(EMA\)\. The associated calls include: describeChannel, createChannel, getChannelEndpoint and deleteChannel\. | 
| dpApiCallLatency | Calculate latency for data plan API calls\. Calculation is done using Exponential Moving Average \(EMA\)\. The associated calls include: getIceConfig\. | 
| signalingClientUptime | This indicates the time for which the client object exists\. Every time this metric is invoked, the most recent uptime value is emitted\. | 
| connectionDuration | If connection is established, this emits the duration for which the connection is alive\. Else, a value of 0 is emitted\. This is different from signaling client uptime since, connections come and go, but signalingClientUptime is indicative of the client object itself\. | 
| numberOfMessagesSent | This value is updated when the peer sends an offer, answer, or an ICE candidate\. | 
| numberOfMessagesReceived | Unlike numberOfMessagesSent, this metric is updated for any type of signaling message\. The types of signaling messages are available in SIGNALING\_MESSAGE\_TYPE\. | 
| iceRefreshCount | This is incremented when getIceConfig is invoked\. The rate at which this is invoked is based on the TTL as part of the ICE configuration received\. Each time a fresh set of ICE configuration is received, a timer is set to refresh next time, given the validity of the credentials in the configuration minus some grace period\.  | 
| numberOfErrors | The counter is used to track the number of errors generated within the signaling client\. Errors generated while getting ICE configuration, getting signaling state, tracking signaling metrics, sending signaling message, and connecting the signaling client to the web socket in order to send/receive messages are tracked\. | 
| numberOfRuntimeErrors | The metric includes errors that are incurred while the core of the signaling client is running\. Scenarios like reconnect failures, message receive failures, and ICE configuration refresh errors are tracked here\. | 
| numberOfReconnects | The metric is incremented on every reconnect\. This is a useful metric to understand the stability of the network connection in the set up\. | 

## W3C Standard Metrics Supported for WebRTC C SDK<a name="kvswebrtc-reference-w3cstandard"></a>

A subset of the [W3C](https://www.w3.org/TR/webrtc-stats/) standard metrics is currently supported for the applications built with the WebRTC C SDK\. These fall into the following categories:
+ Networking:
  + [Ice Candidate](https://www.w3.org/TR/webrtc-stats/#icecandidate-dict*): these metrics provide information about the selected local and remote candidates for data exchange between the peers\. This includes server source of the candidate, IP address, type of candidate selected for the communication, and candidate priority\. These metrics are useful as a snapshot report\.
  + [Ice Server](https://www.w3.org/TR/webrtc-stats/#ice-server-dict*): these metrics are for gathering operational information about the different ICE servers supported\. This is useful when trying to understand the server that is primarily being used for communication and connectivity checks\. In some instances, it is also useful to examine these metrics if the gathering of candidates fails\. 
  + [Ice Candidate Pair](https://www.w3.org/TR/webrtc-stats/#candidatepair-dict*): these metrics are for understanding the number of bytes/packets that are being exchanged between the peers and also time\-related measurements\.
+ Media and data:
  + [Remote Inbound RTP](https://www.w3.org/TR/webrtc-stats/#remoteinboundrtpstats-dict*): these metrics represent the endpoint perspective of the data stream sent by the sender\.
  + [Outbound RTP](https://www.w3.org/TR/webrtc-stats/#dom-rtcoutboundrtpstreamstats): these metrics provide information about the outgoing RTP stream\. They can also be very useful when analyzing choppy streaming or streaming stops\.
  + [Inbound RTP](https://www.w3.org/TR/webrtc-stats/#dom-rtcinboundrtpstreamstats): these metrics provide information about the incoming media\. 
  + [Data channel metrics](https://www.w3.org/TR/webrtc-stats/#dcstats-dict*): these metrics can help you analyze the number of messages and bytes sent and received over a data channel\. The metrics can be pulled by using the channel ID\.

You can use the `STATUS rtcPeerConnectionGetMetrics (PRtcPeerConnection, PRtcRtpTransceiver, PRtcStats)` API to gather metrics related to ICE, RTP and the data channel\. Here's a usage example:

```
RtcStats rtcStats;
rtcStats.requestedTypeOfStats = RTC_STATS_TYPE_LOCAL_CANDIDATE;
STATUS retStatus = rtcPeerConnectionGetMetrics (pRtcPeerConnection, NULL, &rtcStats);
printf(“Local Candidate address: %s\n”, rtcStats.rtcStatsObject.localIceCandidateStats.address);
```

Here's another example that shows usage pattern to get transceiver related stats:

```
RtcStats rtcStats;
PRtcRtpTransceiver pVideoRtcRtpTransceiver;
rtcStats.requestedTypeOfStats = RTC_STATS_TYPE_OUTBOUND_RTP;
STATUS retStatus = rtcPeerConnectionGetMetrics (pRtcPeerConnection, pVideoRtcRtpTransceiver, &rtcStats);
printf(“Number of packets discarded on send: %s\n”, rtcStats.rtcStatsObject.outboundRtpStreamStats.packetsDiscardedOnSend);
```

In the above example, if the second argument to rtcPeerConnectionGetMetrics\(\) is NULL, data for the first transceiver in the list is returned\.

Definition for rtcStatsObject can be found in [Stats\.h](https://github.com/awslabs/amazon-kinesis-video-streams-webrtc-sdk-c/blob/master/src/include/com/amazonaws/kinesis/video/webrtcclient/Stats.h)\. and definition for RtcStats can be found in [Include\.h](https://github.com/awslabs/amazon-kinesis-video-streams-webrtc-sdk-c/blob/master/src/include/com/amazonaws/kinesis/video/webrtcclient/Include.h)\. 

Sample usages of the APIs and the different metrics can be found in the [samples](https://github.com/awslabs/amazon-kinesis-video-streams-webrtc-sdk-c/tree/master/samples) directory in the WebRTC C SDK repository and in the [Kinesis Video Stream demos repository](https://github.com/aws-samples/amazon-kinesis-video-streams-demos/tree/master/webrtc-c/canary/src)\.

The following [W3C](https://www.w3.org/TR/webrtc-stats/) standard metrics are currently supported for the applications built with the WebRTC C SDK\.

**Topics**
+ [Networking](#kvswebrtc-reference-ice)
+ [Media](#kvswebrtc-reference-media)
+ [Data Channel](#kvswebrtc-reference-datachannel)

### Networking<a name="kvswebrtc-reference-ice"></a>

ICE Server Metrics:


****  

| Metric | Description | 
| --- | --- | 
| URL | URL of the STUN/TURN server being tracked | 
| Port | Port number used by the client | 
| Protocol | Transport protocol extracted from ICE Server URI\. If the value is UDP, ICE tries TURN over UDP, else ICE tried TURN over TCP/TLS\. If the URI does not contain transport, ICE tries TURN over UDP and TCP/TLS\. In case of STUN server, this field is empty\. | 
| Total Requests Sent  | The value is updated for every srflx candidate request and while sending binding request from turn candidates\. | 
| Total Responses Received | The value is updated every time a STUN binding response is received\. | 
| Total Round Trip Time | The value is updated every time an equivalent response is received for a request\. The request packet is tracked in a hash map with the checksum as the key\. | 

ICE Candidate Stats: Only the information about the selected candidate \(local and remote\) is included\.


****  

| Metric | Description | 
| --- | --- | 
| address | This indicates the IP address of the local and remote candidate\. | 
| port | Port number of the candidate | 
| protocol | Protocol used to obtain the candidate\. The valid values are UDP/TCP\. | 
| candidateType | Type of candidate selected \- host, srflx or relay\. | 
| priority | Priority of the selected local and remote candidate\. | 
| url | Source of the selected local candidate\. This gives an indication of if the candidate selected is received from a STUN server or TURN server\. | 
| relayProtocol | If TURN server is used to obtain the selected local candidate, this field indicates what protocol was used to obtain it\. Valid values are TCP/UDP\. | 

ICE Candidate Pair Stats: Only the information about the selected candidate pairs is included\.


****  

| Metric | Description | 
| --- | --- | 
| localCandidateId | The ID of the selected local candidate in the pair\. | 
| remoteCandidateId | The ID of the selected remote candidate in the pair\.  | 
| state | State of the candidate pair being inspected\. | 
| nominated | Set to TRUE since the stats are being pulled for selected candidate pair\. | 
| packetsSent | Number of packets sent\. This is calculated in the \. call in the writeFrame call\. This information can also be extracted from outgoing RTP Stats, but since Ice candidate pair includes a lastPacketSent timestamp, it might be useful to calculate number of packets sent between two points in time\. | 
| packetsReceived | This is updated every time the incomingDataHandler is called\. | 
| bytesSent | This is calculated in the iceAgentSendPacket\(\) in the writeFrame\(\) call\. This is useful when calculating a bit rate\. Currently, this also includes the header and padding since the ICE layer is oblivious to the RTP packet format\. | 
| bytesReceived | This is updated every time the incomingDataHandler is called\. Currently, this also includes the header and padding since the ICE layer is oblivious to the RTP packet format\. | 
| lastPacketSentTimestamp | This is updated every time a packet is sent\. This can be used in conjunction with the packetsSent and a recorded start time in application to current packet transfer rate\. | 
| lastPacketReceivedTimestamp | This is updated on receiving data in incomingDataHandler\(\)\. This can be used in conjunction with packetsReceived to deduce the current packet receive rate\. The start time has to be recorded at the application layer in the transceiverOnFrame\(\) callback\. | 
| firstRequestTimestamp | Recorded when the very first STUN binding request is sent out successfully in iceAgentSendStunPacket\(\)\. This can be used along with lastRequestTimestamp and requestsSent to find average time between STUN binding requests\. | 
| lastRequestTimestamp | Recorded every time a STUN binding request is sent out successfully in iceAgentSendStunPacket\(\)\. | 
| lastResponseTimestamp | Recorded every time a STUN binding response is received\. | 
| totalRoundTripTime | Updated when a binding response is received for a request\. The request and response are mapped in a hash table based on checksum\.  | 
| currentRoundTripTime | Most recent round trip time updated when a binding response is received for a request on the candidate pair\.  | 
| requestsReceived | A counter that is updated on every STUN binding request received\. | 
| requestsSent | A counter that is updated on every STUN binding request sent out in iceAgentSendStunPacket\(\)\.  | 
| responsesSent | A counter that is updated on every STUN binding response sent out in response to a binding request in handleStunPacket\(\)\. | 
| responsesReceived | A counter that is updated on every STUN binding response received in handleStunPacket\(\)\.  | 
| packetsDiscardedOnSend |  Updated when packet sending fails\. In other words, this is updated when iceUtilsSendData\(\) fails\. This isuseful to determine percentage of packets dropped in a specific duration\. | 
| bytesDiscardedOnSend | Updated when packet sending fails\. In other words, this is updated when iceUtilsSendData\(\) fails\. This is useful when determining percentage of packets dropped in a specific duration\. Note that the counter also includes the header of the packets\. | 

### Media<a name="kvswebrtc-reference-media"></a>

Outbound RTP Stats


****  

| Metric | Description | 
| --- | --- | 
| voiceActivityFlag | This is currently part of RtcEncoderStats defined in Include\.h\. The flag is set to TRUE if the last audio packet contained voice\. The flag is currently not set in the samples\. | 
| packetsSent | This indicates the total number of RTP packets sent out for the selected SSRC\. This is a part of [https://www\.w3\.org/TR/webrtc\-stats/\#sentrtpstats\-dict\*](https://www.w3.org/TR/webrtc-stats/#sentrtpstats-dict*) and is included as part of outbound stats\. This is incremented every time writeFrame\(\) is called\. | 
| bytesSent | Total number of bytes excluding RTP header and padding that is sent\. This is updated on every writeFrame call\. | 
| encoderImplementation | This is updated by the application layer as part of RtcEncoderStats object\.  | 
| packetsDiscardedOnSend | This field is updated if the ICE agent fails to send the encrypted RTP packet for any reason in the iceAgentSendPacket call\. | 
| bytesDiscardedOnSend | This field is also updated if the ICE agent fails to send the encrypted RTP packet for any reason in the iceAgentSendPacket call\. | 
| framesSent | This is incremented only if media stream tack type is MEDIA\_STREAM\_TRACK\_KIND\_VIDEO\. | 
| hugeFramesSent | This counter is updated for frames that are 2\.5 times the average size of frames\. The size of the frame is obtained by calculating the fps \(based on the last known frame count time and number of frames encoded in a time interval\) and using the targetBitrate in RtcEncoderStats set by the application\.  | 
| framesEncoded | This counter is updated only for video track after successful encoding of the frame\. It is updated on every writeFrame call\. | 
| keyFramesEncoded | This counter is updated only for video track after successful encoding of the key frame\. It is updated on every writeFrame call\. | 
| framesDiscardedOnSend | This is updated when frame sending fails due to iceAgentSendPacket call failure\. A frame comprises of a group of packets and currently, framesDiscardedOnSend fails if any packet gets discarded on while sending because of an error\. | 
| frameWidth | This ideally represents the frame width of the last encoded frame\. Currently, this is set to a value by the application as part of RtcEncoderStats\* \*and is of not much significance\. | 
| frameHeight | This ideally represents the frame height of the last encoded frame\. Currently, this is set to a value by the application as part of RtcEncoderStats and is of not much significance\. | 
| frameBitDepth |  This represents the bit depth per pixel width of the last encoded frame\. Currently, this is set by the application as part of RtcEncoderStats and translated into outbound stats\.  | 
| nackCount | This value is updated every time a NACK is received on an RTP packet and a re\-attempt to send the packet is made\. The stack supports re\-transmission of packets on receiving a NACK\. | 
| firCount | The value is updated on receiving a FIR packet \(onRtcpPacket\->onRtcpFIRPacket\)\. It indicates how often the stream falls behind and has to skip frames in order to catch up\. FIR packet is currently not decoded to extract the fields, so, even though the count is set, no action is taken\. | 
| pliCount | The value is updated on receiving a PLI packet \(onRtcpPacket\->onRtcpPLIPacket\)\. It indicates that some amount of encoded video data has been lost for one or more frames\. | 
| sliCount | The value is updated on receiving a SLI packet \(onRtcpPacket\->onRtcpSLIPacket\)\. It indicates how often packet loss affects a single frame\. | 
| qualityLimitationResolutionChanges | Currently, the stack supports this metric, however, the frame width and height are not monitored for every encoded frame\. | 
| lastPacketSentTimestamp | The timestamp at which the last packet was sent\. It is updated on every writeFrame call\. | 
| headerBytesSent | Total number of RTP header and padding bytes sent for this SSRC excluding the actual RTP payload\. | 
| bytesDiscardedOnSend | This is updated when frame sending fails due to iceAgentSendPacket call failure\. A frame comprises of a group of packets, which in turn comprises of bytes and currently, bytesDiscardedOnSend fails if any packet gets discarded on while sending because of an error\. | 
| retransmittedPacketsSent | The number of packets that are retransmitted on reception of PLI/SLI/NACK\. Currently, the stack only counts the packet resent of NACK since PLI and SLI based retransmissions are not supported\. | 
| retransmittedBytesSent | The number of bytes that are retransmitted on reception of PLI/SLI/NACK\. Currently, the stack only counts the bytes resent of NACK since PLI and SLI based retransmissions are not supported\. | 
| targetBitrate | This is set in the application level\. | 
| totalEncodedBytesTarget | This is increased by the target frame size in bytes every time a frame is encoded\. This is updated using size parameter in Frame structure\. | 
| framesPerSecond | This is calculated based on the time recorded for the last known encoded frame and the number of frames sent within a second\. | 
| totalEncodeTime | This is set to an arbitrary value in the application and is translated to outbound stats internally\. | 
| totalPacketSendDelay | This is currently set to 0 since iceAgentSendPacket sends packet immediately\. | 

Remote inbound RTP Stats:


****  

| Metric | Description | 
| --- | --- | 
| roundTripTime | The value is extracted from the RTCP receiver report on receiving an RTCP packet type 201 \(receiver report\)\. The report comprises of details such as last sender report and delay since last sender report to calculate round trip time\. Sender reports are generated roughly every 200 milliseconds comprising of information such as number of packets sent and bytes sent that are extracted from outbound stats\. | 
| totalRoundTripTime | Sum of round trip times calculated | 
| fractionLost | Represents the fraction of RTP packets lost for the SSRC since the previous sender/receiver reporfractionLost was sent\. | 
| reportsReceived | Updated every time a receiver report type packet is received\. | 
| roundTripTimeMeasurements | Indicates the total number of reports received for the SSRC that contains valid round trip time\. However, currently this value is incremented regardless so its meaning is the same as reportsReceived\. | 

Inbound RTP Stats:


****  

| Metric | Description | 
| --- | --- | 
| packetsReceived | The counter is updated when a packet is received for a specific SSRC\. | 
| jitter | This metric indicates the packet Jitter measured in seconds for the specific SSRC\. | 
| jitterBufferDelay | This metric denotes the sum of time spent by each packet in the jitter buffer\. | 
| jitterBufferEmittedCount | The total number of audio samples or video frames that have come out of the jitter buffer\.  | 
| packetsDiscarded | The counter is updated when the Jitter buffer is full and the packet cannot be pushed into it\. This can be used to calculate percentage of packets discarded in a fixed duration\. | 
| framesDropped | This value is updated when the onFrameDroppedFunc\(\) is invoked\.  | 
| lastPacketReceivedTimestamp | Represents the timestamp at which the last packet was received for this SSRC\. | 
| headerBytesReceived | The counter is updated on receiving an RTP packet\. | 
| bytesReceived | Number of bytes received\. This does not include the header bytes\. This metric can be used to calculate the incoming bit rate\. | 
| packetsFailedDecryption | This is incremented when the decryption of the SRTP packet fails\. | 

### Data Channel<a name="kvswebrtc-reference-datachannel"></a>

Data Channel Metrics


****  

| Metric | Description | 
| --- | --- | 
| label | Label is the name of the data channel being inspected\. | 
| protocol | Since our stack uses SCTP, the protocol is set to a constant SCTP\. | 
| dataChannelIdentifier | The even or odd identifier used to uniquely identify a data channel\. This is updated to an odd value if the SDK is the offerer and even value if SDK is the answerer\. | 
| state | State of the data channel when the stats are queried\. Currently, the two states supported are RTC\_DATA\_CHANNEL\_STATE\_CONNECTING \(when the channel is created\) and RTC\_DATA\_CHANNEL\_STATE\_OPEN \(Set in the onOpen\(\) event\)\. | 
| messagesSent | The counter is updated when the SDK sends messages over the data channel\. | 
| bytesSent | The counter is updated with the bytes in the message that is sent out\. This can be used to understand how many bytes are not sent due to failure, i\.e, to understand the percentage of bytes that are sent\. | 
| messagesReceived | The metric is incremented in the onMessage\(\) callback\. | 
| bytesReceived | The metric is generated in the onMessage\(\) callback\. | 