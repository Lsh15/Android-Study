# Singleton
Singleton(싱글톤) 패턴이란 인스턴스를 1개만 생성하여 재사용하는 패턴이다.

## 장점
*	메모리 낭비 방지   
고정된 메모리 영역을 사용하기 때문에 추후 해당 객체에 접근할 때 메모리 낭비를 방지할 수 있다.’
*	데이터 공유 쉬움    
전역으로 사용되는 인스턴스이기 때문에 다른 클래스의 인스턴스들이 쉽게 접근하여 사용할 수 있다.

## 단점
*	테스트의 어려움     
Singleton(싱글톤) 인스턴스는 자원을 공유하고 있기 때문에 테스트가 결정적으로 격리된 환경에서 수행되려면 매번 인스턴스의 상태를 초기화시켜주어야 한다. 그렇지 않으면 어플리케이션 전역에서 상태를 공유하기 때문에 테스트가 온전하게 수행되지 못한다.
*	개방-폐쇄 원칙 위배    
Singleton(싱글톤) 인스턴스가 혼자 너무 많은 일을 하거나, 많은 데이터를 공유시키면 다른 클래스들 간의 결합도가 높아지게 되는데, 이때 개방-폐쇄 원칙이 위배된다.
결합도가 높아지게 되면, 유지보수가 힘들고 테스트도 원활하게 진행할 수 없는 문제점이 발생한다.

## object
클래스 이름 앞에 object 키워드를 붙이면 Singleton(싱글톤) 클래스를 만들수 있다.   
생성자를 호출하지 않는 클래스에서만 사용할 수 있다.
``` kotlin
object ExSingletonClass{ }
val exSingleton = ExSingletonClass
```

## companion object
생성자를 통해 파라메터를 전달받는 Singleton(싱글톤) 클래스를 만들기 위해선 companion object 를 사용한다.
``` kotlin
class ExSingletonClass private constructor(context: Context) {

    companion object {
    
    //자기 자신 변수 가져오기
        @Volatile
        private var instance: ExSingletonClass? = null
    
    //자기 자신 가져오기
        fun getInstance(context: Context): ExSingletonClass =
            instance ?: synchronized(this) {
                instance ?: ExSingletonClass(context).also {
                    instance = it
                }
            }
        }
    }
```


### 참고
https://tecoble.techcourse.co.kr/post/2020-11-07-singleton/    
https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80%ED%84%B4_%ED%8C%A8%ED%84%B4     
https://bacassf.tistory.com/59     
