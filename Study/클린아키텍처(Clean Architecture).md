# 클린 아키텍처(Clean Architecture)

## 1. 클린 아키텍처란?
클린 아키텍처는 **애플리케이션의 핵심 규칙(비즈니스 로직)**을 UI, DB, 네트워크, 프레임워크 등 **외부 요소로부터 분리**하여,  
변경에 강하고 테스트하기 쉬운 구조를 만드는 설계 원칙입니다.

**핵심 원칙**
- **Dependency Rule(의존성 규칙)**: 모든 의존성은 항상 **안쪽(도메인)**을 향해야 합니다.
- **경계 분리**: 계층 간 데이터 구조는 독립적으로 두고, 변환은 매퍼(Mapper)에서 수행.
- **순수성 보장**: 도메인 로직은 외부 기술이나 라이브러리에 의존하지 않는 **순수 코드**여야 함.

---

## 2. 왜 필요하게 되었나?
과거 안드로이드 앱 구조는 **Activity/Fragment가 UI·로직·데이터를 모두 처리**하는 형태가 많았습니다.  
이 경우:
- UI 변경이 로직/데이터 코드까지 줄줄이 영향을 미침
- 테스트가 어려움 (UI 없이 로직만 검증 불가)
- 네트워크·DB 교체 시 수정 범위가 커짐

클린 아키텍처는 이런 문제를 해결합니다:
- **변경 내성**: DB → API 변경, UI 프레임워크 교체 등에도 도메인 로직 불변
- **테스트 용이**: 안드로이드 환경 없이 순수 로직 단위 테스트 가능
- **확장성**: 새로운 UI나 데이터 소스 추가가 쉬움

---

## 3. 계층 구조
보통 3~4계층으로 나누며, 의존 방향은 항상 바깥 → 안쪽입니다.

1. **Entities (도메인 모델)**  
   - 핵심 비즈니스 개념/규칙
2. **Use Cases (애플리케이션 규칙)**  
   - 특정 시나리오(입력→출력) 구현
3. **Interface Adapters**  
   - 데이터 변환, Repository 구현, ViewModel, Mapper
4. **Frameworks & Drivers**  
   - UI, DB, 네트워크, DI, OS 등 외부 기술

---

## 4. 이 프로젝트의 멀티모듈 구조 매핑
- **`presentation/`**: View, ViewModel, UI 상태/이벤트, 화면 렌더링
- **`domain/`**: Entities, UseCases, Repository 인터페이스
- **`data/`**: Repository 구현, DataSource(API/DB), DTO, Mapper

**의존 방향**
```
presentation → domain ← data
```
- `presentation` ↛ `data` 직접 의존 없음 (의존 역전)

---

## 5. 실전 적용 예시

### Domain Layer (`:domain`)
- **구성**: UseCase, Repository 인터페이스, Entity
- **예시 – SearchVideoUseCase**
```kotlin
class SearchVideoUseCase @Inject constructor(
    private val videoRepository: VideoRepository
) {
    suspend operator fun invoke(query: String) =
        videoRepository.getVideosPager(query)
}
```

---

### Data Layer (`:data`)
- **구성**: API, DB, DTO, Mapper, Repository 구현체
- **예시 – Repository 구현**
```kotlin
class VideoRepositoryImpl @Inject constructor(
    private val videoApi: VideoApi,
    private val videoDao: VideoDao
) : VideoRepository {
    override suspend fun getVideosPager(query: String) =
        Pager(PagingConfig(pageSize = 10)) {
            VideoPagingSource(query, videoApi)
        }.flow
}
```
- **Mapper 예시**
```kotlin
fun VideoEntity.toVideoLocalItem() = VideoLocalItem(id, title, thumbnail)
```

---

### Presentation Layer (`:presentation`)
- **구성**: Activity/Fragment, ViewModel, UI State(Event), 수집 로직
- **예시 – ViewModel**
```kotlin
@HiltViewModel
class SearchVideoViewModel @Inject constructor(
    private val searchVideoUseCase: SearchVideoUseCase
) : ViewModel() {
    val query = MutableStateFlow("")
    val pagedVideos = query
        .filter { it.isNotBlank() }
        .flatMapLatest { searchVideoUseCase(it) }
        .cachedIn(viewModelScope)
}
```
- **Fragment 수집**
```kotlin
repeatOnLifecycle(Lifecycle.State.STARTED) {
    launch { viewModel.pagedVideos.collectLatest { adapter.submitData(it) } }
}
```

---

### DI 결선
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object RepositoryModule {
    @Provides
    fun provideVideoRepository(api: VideoApi, dao: VideoDao): VideoRepository =
        VideoRepositoryImpl(api, dao)
}
```

---

## 6. 장단점

**장점**
- 변경에 강함 (UI/DB/네트워크 교체 시 영향 최소)
- 테스트 용이
- 모듈/책임 분리로 가독성, 유지보수성 ↑

**단점**
- 보일러플레이트 증가
- 작은 프로젝트에는 과설계 가능
- 팀 합의 없이 부분 적용 시 복잡도 증가

---

## 7. 운영 팁
- 상태는 `StateFlow`, 이벤트는 `SharedFlow`, 계산/쿼리는 `Flow`
- API/DB 변경은 `:data` DTO/Mapper만 수정
- DI 모듈에서 인터페이스-구현 결선을 중앙 관리
- `repeatOnLifecycle`로 라이프사이클 안전하게 수집

---

## 8. 결론
클린 아키텍처는 **도메인 규칙을 중심에 두고, 외부 기술 의존성을 경계 밖으로 밀어낸 구조**입니다.  
이 프로젝트에서는 `presentation`, `domain`, `data` 모듈로 나누고,  
DI·Mapper·Paging 등을 활용해 변경 내성·테스트 용이성을 실전 구현했습니다.

**요약**
- 구조: `presentation` → `domain` ← `data`
- 핵심: 의존성은 안쪽 향함, 경계는 매퍼로 연결, 도메인은 순수하게
- 효과: 유지보수성·확장성·테스트 용이성 확보
