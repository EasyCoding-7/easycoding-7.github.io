---
layout: post
title:  "(OpenSource : WebRTC) 임시 Note"
summary: ""
author: OpenSource
date: '2021-05-26 0:00:00 +0000'
category: ['OpenSource', 'WebRTC']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/opensource-thumnail.png
#keywords: ['unique_ptr']
usemathjax: true
permalink: /blog/opensource/WebRTC/temp-note/
---

## Client

```cpp
void MainWnd::OnDefaultAction() {
  if (!callback_)
    return;
  if (ui_ == CONNECT_TO_SERVER) {
    std::string server(GetWindowText(edit1_));
    std::string port_str(GetWindowText(edit2_));
    int port = port_str.length() ? atoi(port_str.c_str()) : 0;
    callback_->StartLogin(server, port);                        // 1. Server와 연결
  } else if (ui_ == LIST_PEERS) {
    LRESULT sel = ::SendMessage(listbox_, LB_GETCURSEL, 0, 0);
    if (sel != LB_ERR) {
      LRESULT peer_id = ::SendMessage(listbox_, LB_GETITEMDATA, sel, 0);
      if (peer_id != -1 && callback_) {
        callback_->ConnectToPeer(peer_id);                      // 2. Peer와 연결
      }
    }
  } else {
    ::MessageBoxA(wnd_, "OK!", "Yeah", MB_OK);
  }
}
```

### 1. Server와 연결

```cpp
void Conductor::StartLogin(const std::string& server, int port) {
  if (client_->is_connected())
    return;
  server_ = server;
  client_->Connect(server, port, GetPeerName());
}
```

```cpp
void PeerConnectionClient::Connect(const std::string& server,
                                   int port,
                                   const std::string& client_name) {
  RTC_DCHECK(!server.empty());
  RTC_DCHECK(!client_name.empty());

  if (state_ != NOT_CONNECTED) {
    RTC_LOG(WARNING)
        << "The client must not be connected before you can call Connect()";
    callback_->OnServerConnectionFailure();
    return;
  }

  if (server.empty() || client_name.empty()) {
    callback_->OnServerConnectionFailure();
    return;
  }

  if (port <= 0)
    port = kDefaultServerPort;

  server_address_.SetIP(server);
  server_address_.SetPort(port);
  client_name_ = client_name;

  if (server_address_.IsUnresolvedIP()) {
    state_ = RESOLVING;
    resolver_ = new rtc::AsyncResolver();
    resolver_->SignalDone.connect(this, &PeerConnectionClient::OnResolveResult);
    resolver_->Start(server_address_);
  } else {
    DoConnect();
  }
}
```

### 2. Peer와 연결

```cpp
void Conductor::ConnectToPeer(int peer_id) {
  RTC_DCHECK(peer_id_ == -1);
  RTC_DCHECK(peer_id != -1);

  if (peer_connection_.get()) {
    main_wnd_->MessageBox(
        "Error", "We only support connecting to one peer at a time", true);
    return;
  }

  if (InitializePeerConnection()) {     // 2-1. Peer 초기화
    peer_id_ = peer_id;
    peer_connection_->CreateOffer(      // 2-2. CreateOffer 생성
        this, webrtc::PeerConnectionInterface::RTCOfferAnswerOptions());
  } else {
    main_wnd_->MessageBox("Error", "Failed to initialize PeerConnection", true);
  }
}
```

#### 2-1. Peer 초기화

```cpp
bool Conductor::InitializePeerConnection() {
  RTC_DCHECK(!peer_connection_factory_);
  RTC_DCHECK(!peer_connection_);

  peer_connection_factory_ = webrtc::CreatePeerConnectionFactory(
      nullptr /* network_thread */, nullptr /* worker_thread */,
      nullptr /* signaling_thread */, nullptr /* default_adm */,
      webrtc::CreateBuiltinAudioEncoderFactory(),
      webrtc::CreateBuiltinAudioDecoderFactory(),
      webrtc::CreateBuiltinVideoEncoderFactory(),
      webrtc::CreateBuiltinVideoDecoderFactory(), nullptr /* audio_mixer */,
      nullptr /* audio_processing */);

  if (!peer_connection_factory_) {
    main_wnd_->MessageBox("Error", "Failed to initialize PeerConnectionFactory",
                          true);
    DeletePeerConnection();
    return false;
  }

  if (!CreatePeerConnection(/*dtls=*/true)) {
    main_wnd_->MessageBox("Error", "CreatePeerConnection failed", true);
    DeletePeerConnection();
  }

  AddTracks();

  return peer_connection_ != nullptr;
}
```

#### 2-2. CreateOffer 생성

```cpp
  // Create a new offer.
  // The CreateSessionDescriptionObserver callback will be called when done.
  virtual void CreateOffer(CreateSessionDescriptionObserver* observer,
                           const RTCOfferAnswerOptions& options) = 0;

// 옵저버와 옵션을 등록함.
```