# Sleep App

iOS 수면 녹음 앱 — 수면 중 오디오를 녹음하고, 타임라인과 파형으로 결과를 확인할 수 있습니다.

## 주요 기능

- **녹음** — 원터치 녹음 시작/중지, 백그라운드 녹음 지원
- **녹음 목록** — 24시간 타임라인 차트에서 녹음 기록 한눈에 확인
- **상세 재생** — 오디오 파형 시각화, 재생/일시정지/탐색(10초 스킵)

## 기술 스택

| 구분 | 기술 |
|------|------|
| UI | SwiftUI, Swift Charts |
| 오디오 | AVFoundation (AVAudioRecorder, AVAudioPlayer) |
| 상태 관리 | @Observable (iOS 17+), Combine |
| 아키텍처 | Clean Architecture (Domain / Infra / Presentation) |
| DI | AppDependencyContainer + @Environment |
| 데이터 영속화 | JSON 파일 (Documents/Recordings/) |
| 최소 지원 | iOS 18.0+ |

## 아키텍처

```
┌─────────────────────────────────────────────┐
│  App                                        │
│  └─ DI Container → ViewModel 팩토리          │
├─────────────────────────────────────────────┤
│  Presentation                               │
│  ├─ Recording (View + ViewModel)            │
│  ├─ RecordingList (View + ViewModel)        │
│  └─ RecordingDetail (View + ViewModel)      │
├─────────────────────────────────────────────┤
│  Domain                                     │
│  ├─ Models (Recording, RecordingState, ...)  │
│  └─ Protocols (AudioRecorder, Player, ...)   │
├─────────────────────────────────────────────┤
│  Infra                                      │
│  ├─ Audio (Session, Recorder, Player)        │
│  └─ Persistence (FileManager, Repository)    │
└─────────────────────────────────────────────┘
```

의존성 방향: `Presentation → Domain ← Infra`

## 프로젝트 구조

```
Asleep_pre/
├── App/
│   ├── Asleep_preApp.swift
│   └── DI/
│       └── AppDependencyContainer.swift
├── Domain/
│   ├── Models/
│   │   ├── Recording.swift
│   │   ├── RecordingState.swift
│   │   ├── PlaybackState.swift
│   │   ├── AudioQuality.swift
│   │   └── MeteringLevel.swift
│   └── Protocols/
│       ├── AudioSessionProtocol.swift
│       ├── AudioRecorderProtocol.swift
│       ├── AudioPlayerProtocol.swift
│       └── RecordingRepositoryProtocol.swift
├── Infra/
│   ├── Audio/
│   │   ├── AudioSessionService.swift
│   │   ├── AudioRecorderService.swift
│   │   └── AudioPlayerService.swift
│   └── Persistence/
│       ├── RecordingFileManager.swift
│       └── RecordingRepository.swift
├── Presentation/
│   ├── ContentView.swift
│   ├── Recording/
│   │   ├── RecordingView.swift
│   │   ├── RecordingViewModel.swift
│   │   └── Components/
│   │       ├── RecordButton.swift
│   │       └── RecordingTimerView.swift
│   ├── RecordingList/
│   │   ├── RecordingListView.swift
│   │   └── RecordingListViewModel.swift
│   └── RecordingDetail/
│       ├── RecordingDetailView.swift
│       ├── RecordingDetailViewModel.swift
│       └── Components/
│           ├── WaveformChartView.swift
│           ├── PlaybackControlView.swift
│           └── PlaybackSliderView.swift
├── Utils/
│   ├── Constants/
│   │   └── AppTheme.swift
│   └── Extensions/
│       ├── DateFormatter+Extensions.swift
│       └── TimeInterval+Extensions.swift
└── Resources/
    └── Assets.xcassets/
```

## 데이터 흐름

```
사용자 액션 → View → ViewModel → Service (AVFoundation)
                         ↑              │
                         └── Combine ───┘
                              (Publisher/Subscriber)
```

- **녹음**: `RecordingViewModel` → `AudioRecorderService` → Combine으로 상태/미터링 실시간 수신
- **재생**: `RecordingDetailViewModel` → `AudioPlayerService` → Combine으로 재생 상태 실시간 수신
- **저장**: `RecordingRepository` → JSON 메타데이터 + M4A 파일 (Documents/Recordings/)

## 빌드 및 실행

1. Xcode 16+ 에서 `Asleep_pre.xcodeproj` 열기
2. 실기기 연결 (마이크 사용을 위해 시뮬레이터 대신 실기기 권장)
3. Signing & Capabilities에서 Team 설정
4. `Cmd + R` 로 빌드 및 실행

> 백그라운드 녹음을 위해 **Background Modes > Audio** 가 활성화되어 있어야 합니다.
