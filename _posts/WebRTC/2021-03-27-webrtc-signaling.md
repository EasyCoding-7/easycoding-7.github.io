---
layout: post
title:  "(WebRTC) Signaling"
summary: ""
author: WebRTC
date: '2021-03-27 0:00:00 +0000'
category: ['WebRTC']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/webrtc-thumnail.webp
#keywords: ['weak_ptr']
usemathjax: true
permalink: /blog/WebRTC/signaling/
---

* [참고사이트 : WebRTC에 대한 전반 설명](https://www.html5rocks.com/ko/tutorials/webrtc/infrastructure/)
* [참고사이트2 : ICE 설명](https://brunch.co.kr/@linecard/156)
* [참고3 : 아직 자세히는 안읽어봄 내용 좋은듯](https://rosypark.tistory.com/m/291?category=726178)

---

## WebRTC Signaling이 어떻게 이루어지는지 Example을 통해서 살펴본다.

1. RTCPeerConnection 객체를 생성
2. createOffer() 메소드를 사용하여 SDP(Session Description Protocol)을 생성
3. SDP와 함께 setLocalDescription()를 호출
4. SDP를 문자열화하고 상대 Peer에게 보냄
5. 수신측에서는 setRemoteDescription()를 통해 상대 Peer의 SDP를 인지
6. 수신측에서는 createAnswer()를 통해 자신의 SDP정보를 전달
7. 상대의 SDP를 받으면 SetRemoteDescription을 통하여 인지

이렇게 말로 적으니 무슨소린지 영 ... Example을 통해 설명해 보자면 ...

### 1. RTCPeerConnection 객체를 생성

뭐.. 간단해서 따로 설명할 부분은 없다.<br>
peerconnection을 만든다

```cpp
bool Conductor::CreatePeerConnection(bool dtls) {
  ASSERT(peer_connection_factory_.get() != NULL);
  ASSERT(peer_connection_.get() == NULL);

  webrtc::PeerConnectionInterface::IceServers servers;
  webrtc::PeerConnectionInterface::IceServer server;
  server.uri = GetPeerConnectionString();
  servers.push_back(server);

  webrtc::FakeConstraints constraints;
  if (dtls) {
    constraints.AddOptional(webrtc::MediaConstraintsInterface::kEnableDtlsSrtp,
                            "true");
  }
  else
  {
    constraints.AddOptional(webrtc::MediaConstraintsInterface::kEnableDtlsSrtp,
                            "false");
  }

  peer_connection_ =
      peer_connection_factory_->CreatePeerConnection(servers,
                                                     &constraints,
                                                     NULL,
                                                     NULL,
                                                     this);
  return peer_connection_.get() != NULL;
}
```

### 2. createOffer() 메소드를 사용하여 SDP(Session Description Protocol)을 생성

```cpp
void Conductor::ConnectToPeer(int peer_id) {
  ASSERT(peer_id_ == -1);
  ASSERT(peer_id != -1);

  if (peer_connection_.get()) {
    main_wnd_->MessageBox("Error",
        "We only support connecting to one peer at a time", true);
    return;
  }

  if (InitializePeerConnection()) { // 여기서 CreatePeerConnection과 Stream을 생성
    peer_id_ = peer_id;
    peer_connection_->CreateOffer(this, NULL);  // CreateOffer
  } else {
    main_wnd_->MessageBox("Error", "Failed to initialize PeerConnection", true);
  }
}
```

### 3. SDP와 함께 setLocalDescription()를 호출
### 4. SDP를 문자열화하고 상대 Peer에게 보냄

```cpp
void Conductor::OnSuccess(webrtc::SessionDescriptionInterface* desc) {
  peer_connection_->SetLocalDescription(
      DummySetSessionDescriptionObserver::Create(), desc);

  // ...

  Json::StyledWriter writer;
  Json::Value jmessage;
  jmessage[kSessionDescriptionTypeName] = desc->type();
  jmessage[kSessionDescriptionSdpName] = sdp;
  SendMessage(writer.write(jmessage));      // sdp를 상대 peer에게 보내게된다.
}
```

참고로 OnSuccess는 CreateOffer에 의해 호출된다.

```cpp
// CreateSessionDescriptionObserver implementation.
virtual void OnSuccess(webrtc::SessionDescriptionInterface* desc);
```

### 5. 수신측에서는 setRemoteDescription()를 통해 상대 Peer의 SDP를 인지
### 6. 수신측에서는 createAnswer()를 통해 자신의 SDP정보를 전달

```cpp
void Conductor::OnMessageFromPeer(int peer_id, const std::string& message) {
  ASSERT(peer_id_ == peer_id || peer_id_ == -1);
  ASSERT(!message.empty());

    // ...

    std::string sdp;
    if (!rtc::GetStringFromJsonObject(jmessage, kSessionDescriptionSdpName,
                                      &sdp)) {
      LOG(WARNING) << "Can't parse received session description message.";
      return;
    }
    webrtc::SdpParseError error;
    webrtc::SessionDescriptionInterface* session_description(
        webrtc::CreateSessionDescription(type, sdp, &error));
    if (!session_description) {
      LOG(WARNING) << "Can't parse received session description message. "
          << "SdpParseError was: " << error.description;
      return;
    }
    LOG(INFO) << " Received session description :" << message;
    peer_connection_->SetRemoteDescription(
        DummySetSessionDescriptionObserver::Create(), session_description);
    if (session_description->type() ==
        webrtc::SessionDescriptionInterface::kOffer) {
      peer_connection_->CreateAnswer(this, NULL);
    }
    return;
  } else {

      // ...
```
