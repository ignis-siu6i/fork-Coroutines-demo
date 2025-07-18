# ApiService

이 파일은 코루틴 테스트에서 사용되는 테스트용 API 서비스 인터페이스를 정의합니다. 실제 네트워크 호출을 시뮬레이션하기 위한 기본 인터페이스로, Mock 객체나 Fake 구현체를 만들 때 사용됩니다.

---

## 인터페이스 정의
```kotlin
interface ApiService {
    /**
     * Get all articles
     */
    suspend fun getArticles(): Resource<List<Article>>

    /**
     * Get the most recommended (i.e., top-ranked) article
     */
    suspend fun getTopArticle(): Resource<Article>

    /**
     * Get all the articles written by a specific author
     */
    fun getArticlesByAuthorName(name: String): LiveData<Resource<List<Article>>>
}
```

---

## 함수별 분석

### getArticles()
```kotlin
suspend fun getArticles(): Resource<List<Article>>
```
**특징:**
- **suspend 함수**: 코루틴 컨텍스트에서 실행
- **Resource 래핑**: 성공/실패 상태를 포함한 응답
- **용도**: 전체 기사 목록 조회 API 시뮬레이션

### getTopArticle()
```kotlin
suspend fun getTopArticle(): Resource<Article>
```
**특징:**
- **단일 객체 반환**: 하나의 Article 객체
- **추천 기사**: 가장 인기 있거나 추천되는 기사
- **용도**: 특별한 컨텐츠 조회 API 시뮬레이션

### getArticlesByAuthorName()
```kotlin
fun getArticlesByAuthorName(name: String): LiveData<Resource<List<Article>>>
```
**특징:**
- **LiveData 반환**: 반응형 데이터 스트림
- **비 suspend 함수**: 즉시 LiveData 객체 반환
- **매개변수**: 작성자 이름으로 필터링
- **용도**: 실시간 데이터 업데이트 시뮬레이션

---

## Resource 타입 설명

### Resource<T> 패턴
```kotlin
sealed class Resource<out T> {
    data class Success<T>(val data: T) : Resource<T>()
    data class Error<T>(val message: String) : Resource<T>()
    data class Loading<T> : Resource<T>()
}
```

**장점:**
- **타입 안전성**: sealed class로 모든 상태 명시
- **명확한 상태**: 성공, 실패, 로딩 상태 구분
- **일관성**: 모든 API 응답에 동일한 패턴 적용

---

## 테스트에서의 사용법

### 1. Mock 구현
```kotlin
@MockK
lateinit var mockApiService: ApiService

@Before
fun setup() {
    MockKAnnotations.init(this)
    
    coEvery { mockApiService.getArticles() } returns Resource.Success(testArticles)
    coEvery { mockApiService.getTopArticle() } returns Resource.Success(topArticle)
    every { mockApiService.getArticlesByAuthorName(any()) } returns MutableLiveData(Resource.Success(authorArticles))
}
```

### 2. Fake 구현
```kotlin
class FakeApiService : ApiService {
    private val articles = mutableListOf<Article>()
    
    override suspend fun getArticles(): Resource<List<Article>> {
        delay(100) // 네트워크 지연 시뮬레이션
        return Resource.Success(articles.toList())
    }
    
    override suspend fun getTopArticle(): Resource<Article> {
        delay(50)
        return if (articles.isNotEmpty()) {
            Resource.Success(articles.first())
        } else {
            Resource.Error("No articles available")
        }
    }
    
    override fun getArticlesByAuthorName(name: String): LiveData<Resource<List<Article>>> {
        val liveData = MutableLiveData<Resource<List<Article>>>()
        val filtered = articles.filter { it.author == name }
        liveData.value = Resource.Success(filtered)
        return liveData
    }
}
```

---

## 실제 구현 시나리오

### Repository 패턴과 함께 사용
```kotlin
class ArticleRepository(
    private val apiService: ApiService,
    private val localDataSource: ArticleDao
) {
    suspend fun refreshArticles(): Resource<List<Article>> {
        return try {
            val result = apiService.getArticles()
            if (result is Resource.Success) {
                localDataSource.insertAll(result.data)
            }
            result
        } catch (e: Exception) {
            Resource.Error(e.message ?: "Unknown error")
        }
    }
    
    fun getArticlesByAuthor(author: String): LiveData<Resource<List<Article>>> {
        return apiService.getArticlesByAuthorName(author)
    }
}
```

---

## 테스트 시나리오 예제

### 성공 케이스 테스트
```kotlin
@Test
fun `getArticles should return success`() = runTest {
    // Given
    val expectedArticles = listOf(
        Article("1", "Author 1", "Title 1"),
        Article("2", "Author 2", "Title 2")
    )
    coEvery { apiService.getArticles() } returns Resource.Success(expectedArticles)
    
    // When
    val result = apiService.getArticles()
    
    // Then
    assertThat(result).isInstanceOf(Resource.Success::class.java)
    val successResult = result as Resource.Success
    assertThat(successResult.data).hasSize(2)
}
```

### 실패 케이스 테스트
```kotlin
@Test
fun `getTopArticle should return error when no articles`() = runTest {
    // Given
    coEvery { apiService.getTopArticle() } returns Resource.Error("No articles found")
    
    // When
    val result = apiService.getTopArticle()
    
    // Then
    assertThat(result).isInstanceOf(Resource.Error::class.java)
    val errorResult = result as Resource.Error
    assertThat(errorResult.message).isEqualTo("No articles found")
}
```

### LiveData 테스트
```kotlin
@Test
fun `getArticlesByAuthorName should return live data`() {
    // Given
    val expectedArticles = listOf(Article("1", "John Doe", "Article"))
    val liveData = MutableLiveData(Resource.Success(expectedArticles))
    every { apiService.getArticlesByAuthorName("John Doe") } returns liveData
    
    // When
    val result = apiService.getArticlesByAuthorName("John Doe")
    
    // Then
    assertThat(result.getValueForTest()).isInstanceOf(Resource.Success::class.java)
}
```

---

## 확장 및 활용

### 인터페이스 확장
```kotlin
interface ExtendedApiService : ApiService {
    suspend fun searchArticles(query: String): Resource<List<Article>>
    suspend fun getArticleDetails(id: String): Resource<ArticleDetail>
    suspend fun updateArticle(article: Article): Resource<Article>
    suspend fun deleteArticle(id: String): Resource<Unit>
}
```

### 실제 네트워크 구현
```kotlin
class RetrofitApiService(
    private val api: ArticleApi
) : ApiService {
    override suspend fun getArticles(): Resource<List<Article>> = try {
        val response = api.getArticles()
        if (response.isSuccessful) {
            Resource.Success(response.body() ?: emptyList())
        } else {
            Resource.Error("HTTP ${response.code()}")
        }
    } catch (e: Exception) {
        Resource.Error(e.message ?: "Network error")
    }
}
```

---

## 결론 및 참고
- **인터페이스 역할:**
  - 테스트와 실제 구현 간의 추상화 제공
  - Mock, Fake, 실제 구현체 교체 가능
  - 일관된 API 계약 정의
- **테스트 활용:**
  - 다양한 응답 시나리오 시뮬레이션
  - 네트워크 지연 및 오류 상황 테스트
  - 의존성 주입을 통한 테스트 격리
- **참고 자료:**
  - [Repository pattern](https://developer.android.com/topic/architecture/data-layer)
  - [Testing with fakes](https://developer.android.com/training/testing/unit-testing/local-unit-tests#fakes) 