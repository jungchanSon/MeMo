# Mock Test
실제 객체들 대신 가짜 객체로 하는 단위 테스트.  
이렇게 가짜 객체로 테스트하면, 복잡한 의존관계를 제거하거나 조정할 수 있음.

# Mock Test를 하는 이유
1. 실제 객체로 테스트하면, 외부와 의존성(DB, Controller 등)을 가져 테스트가 무거워짐(테스트 케이스가 많아질수록 오래걸림)  
2. 또, 실제 객체로 테스트하면 예외 상황을 테스트할 때, 해당 예외 상황을 실제처럼 세팅해줘야함. but, Mcok 테스트는 mock 객체에 stub을 씌울 수 있음 (또, stub을 통해 테스트를 더 직관적으로 할 수 있음)

# Mockito
Spring에서 Mock Test를 편하게 할 수 있도록 지원해주는 테스트 프레임워크
- Mock객체를 쉽게 생성해주는 `mock() `
- Mock객체에 대한 Stub을 쉽게 도와주는 `when()`
- Mock객체의 동작을 검증하는 `verify()` 
- 그 외 애너테이션(`@Mock` `@InjectMocks`) 
등을 지원함

주의할 점
- Mock 객체들은 껍데기 가짜 객체라, 실제 동작을 하지 않음. 따라서, 꼭 mock 객체들의 필요한 동작들에 대해 미리 Stubbing을 해줘야함.

```java
@SpringBootTest // 혹은 @ExtendWith(MockitoExtension.class)class FavoriteWriterTest {  
  
    //Mock 객체 세팅 start    //동작을 테스트 할 객체 = Mock들을 주입받을 객체  
    @InjectMocks  
    private FavoriteWriter favoriteWriter;  
  
    @Mock  
    private FavoriteCommandPort favoriteCommandPort;  
  
    @Mock  
    private FavoriteQueryPort favoriteQueryPort;  
  
    @Mock  
    private FavoriteReader favoriteReader;  
  
    //직접 주입이 필요한 필수 객체는 @Spy로 직접 주입해 생성  
    @Spy  
    private FavoriteMapper favoriteMapper = new FavoriteMapperImpl();  
    //Mock 객체 세팅 end  
    @Test  
    void create() {  
        long userId = 10L;  
  
        //given  
        //필요한 mock 객체와 Stub 세팅  
        FavoriteVO.save favoriteVO = givenFavoriteVOWithId(userId);  
        Favorite favorite = getFavoriteWithUserId(userId);  
  
        //when  
        //테스트할 동작들  
        long savedFavoriteId = favoriteWriter.create(userId, favoriteVO);    
        //테스트 결과로 확인할 동작(verify())과 결과  
        verify(favoriteCommandPort, times(1)).create(favorite);  
        assertEquals(userId, savedFavoriteId);  
    }  
  
    private FavoriteVO.save givenFavoriteVOWithId(long userId) {  
        FavoriteVO.save favoriteVO = mock(FavoriteVO.save.class);  
        FavoriteVO.save save = favoriteVO.of(userId);  
  
        Favorite favorite = favoriteMapper.favoriteVoSaveToDomain(save);  
  
        //when() stub 세팅  
        when(favoriteCommandPort.create(favorite))  
                .thenReturn(userId);  
  
        return favoriteVO;  
    }  
  
    private Favorite getFavoriteWithUserId(long userId) {  
        FavoriteVO.save favoriteVO = mock(FavoriteVO.save.class);  
  
        return favoriteMapper.favoriteVoSaveToDomain(favoriteVO.of(userId));  
    }  
}
```
# BDDMockito
- Mockito에서 제공하는 BDD 테스트 도구. 
- 기본적으론 기본 Mockito와 동일. 
단지, BDD(Behavier-Driven-Development) 테스트 시, given-when-then 과정을 직관적으로 만들어주는 라이브러리  
예를 들어,
- `when()` -> `given()`
- `verify()` -> `then()`
으로 사용할 수 있게 해줌. ( 기능적으론 거의 동일! )

위 테스트 코드를 BDDMockito로 변경 시
```java
@SpringBootTest // 혹은 @ExtendWith(MockitoExtension.class)class FavoriteWriterTest {  
  
    //Mock 객체 세팅 start    //동작을 테스트 할 객체 = Mock들을 주입받을 객체  
    @InjectMocks  
    private FavoriteWriter favoriteWriter;  
  
    @Mock  
    private FavoriteCommandPort favoriteCommandPort;  
  
    @Mock  
    private FavoriteQueryPort favoriteQueryPort;  
  
    @Mock  
    private FavoriteReader favoriteReader;  
  
    //직접 주입이 필요한 필수 객체는 @Spy로 직접 주입해 생성  
    @Spy  
    private FavoriteMapper favoriteMapper = new FavoriteMapperImpl();  
    //Mock 객체 세팅 end  
    @Test  
    void create() {  
        long userId = 10L;  
  
        //given  
        //필요한 mock 객체와 Stub 세팅  
        FavoriteVO.save favoriteVO = givenFavoriteVOWithId(userId);  
        Favorite favorite = getFavoriteWithUserId(userId);  
  
        //when  
        //테스트할 동작들  
        long savedFavoriteId = favoriteWriter.create(userId, favoriteVO);  
  
        //then  
        //테스트 결과로 확인할 동작과 결과  
        then(favoriteCommandPort).should().create(eq(favorite));  
        assertEquals(userId, savedFavoriteId);  
    }  
  
    private FavoriteVO.save givenFavoriteVOWithId(long userId) {  
        FavoriteVO.save favoriteVO = mock(FavoriteVO.save.class);  
        FavoriteVO.save save = favoriteVO.of(userId);  
  
        Favorite favorite = favoriteMapper.favoriteVoSaveToDomain(save);  
  
        //given() stub 세팅  
        given(favoriteCommandPort.create(favorite))  
                .willReturn(userId);  
  
        return favoriteVO;  
    }  
  
    private Favorite getFavoriteWithUserId(long userId) {  
        FavoriteVO.save favoriteVO = mock(FavoriteVO.save.class);  
  
        return favoriteMapper.favoriteVoSaveToDomain(favoriteVO.of(userId));  
    }  
}
```