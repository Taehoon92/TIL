# Mockito

### Quick Summary  
##### What is Mockito?
Mockito is a framework that creates "mock" objects for convenient testing.
##### What is Mock?  
Mock is an object that looks like a real object, but it's controllable by programmer.  

---




### How to make Mock Object?
* **Setting Mockito**  
Add Anotation `@ExtendWith(MockitoExtension.class)` before the test class.
```java
@ExtendWith(MockitoExtension.class)
class SampleTest {
    //...
}
```

* **@Mock**  
A mock object using `@Mock` is fake object and and must stubbing to call a method on it. (Stubbing will be covered in more detail below)  
*<If call a method without stubbing, primative type will return 0, reference type will return null.>*

>For example, there is a UserService class.
>```java
>public class UserService {
>   public User getUser() {
>       return new User("tester");
>   }
>   public int getNum() {
>       return 1;
>   }
>}
>```
>In this test, It used @Mock but did not stubbing, so reference type returns null, primitive type returns 0.
>```java
>@ExtendWith(MockitoExtension.class)
>public class Test {
>   @Mock
>   UserService userService;
>   
>   @Test
>   void testReferenceType() {
>       assertThat(userService.getUser()).isNull(); // return null
>   }
>
>   @Test
>   void testPrimitiveType() {
>       assertThat(userService.getNum).isEqual(0); // return 0
>   }
>}
>```

* **@Spy**  
The mock object using `@Spy`is the real object. When the method is executed, the original value is returned if it didn't stubbing, the stubbing value is returned if it did stubbing. Also, you can selectively stub only the desired object or method.

>For example, in the test without stubbing, the logic of existing object is returned. In the test with stubbing, the stubbing value is returned.
>```java
>@ExtendWith(MockitoExtension.class)
>public class Test {
>   @Spy
>   UserService userService
>   
>   @Test
>   void spyTestWithoutStubbing() {
>       assertThat(userService.getUser()).isEqual("tester"); // return "tester"
>   }
>
>   @Test
>   void spyTestWithStubbing () {
>       User tmpUser = new User("hello");
>       when(userService.getUser()).thenReturn(tmpUser); // Stubbing
>       assertThat(userService.getUser()).isEqual(tmpUser); // return "hello"
>   }
>}

* **@InjectMock**  
`@InjectMock` inject the mock object created with `@Mock` or `@Spy` automatically.
  
<br>
<br>

### Mock Object

Basically Mock object is..
* Nothing happended in the void method.
* Return null if the method has a return value.
* If there is a "primitive type" variable, it takes the default value.
* A Collection makes an empty Collection.

After create the Mock object, we need to make assumptions about how the behaviour under test will proceed. As this time, manipulating the behaviour of the mock object is called **stubbing** or **stub**.  
Briefly summarized, "Stubbing" or "Stub" is <u>defining what return value to return when executing the method of the created mock object.</u>

<br>
<br>
  
### How to do Stubbing in Mockito
There are two ways to do stubbing, using "OngoingStubbing" method and "Stubber" method.

* **OngoingStubbing Method**  
`when({Stubbing Method}).{OngoingStubbing Method}`

| Method | Description |  
|---|---|
|thenReturn | Defines the object returned after calling the stubbing method. |
|thenThrow | Defines the exception thrown after calling the stubbing method. |
|thenAnswer | Defines the action customed after calling the stubbing method.<br><br>***thenReturn or thenThrow method is recommended (mockito javadoc)***
|thenCallRealMethod | call the real method |

Example of the OngoingStubbing Method
```java
public class MemberService {
    public Member getMember() {
        return new Member("tester", "mike");
    }
    public Member getMember(String id, String name) {
        return new Member(id, name);
    }
}

@ExtendWith(MockExtension.class)
public class OngoingStubbingTest() {

    @Mock
    MemberService memberService;

    @Test
    void thenReturnTest() {
        Member member = new Member("hello", "terry");
        when(memberService.getMember()).thenReturn(member);
        assertThat(memberService.getMember()).isEqualTo(member);
    }

    @Test
    void thenThrowTest() {
        when(memberService.getMember()).thenThrow(new IllegalArgumentException());
        assertThatThrownBy(() -> memberService.getMember().isInstanceOf(IllegalArgumentException.class));
    }

    @Test
    void thenAnswerTest() {
        when(memberService.getMember(any(), any())).thenAnswer((Answer) invocation -> {
            Object[] args = invocation.getArguments();
            
            return new Member(args[0]+"AA", args[1]+"BB");
        })

        assertThat(memberService.getMember("hello","terry").getId()).isEqualTo("helloAA");
    }

    @Test
    void thenCallRealMethodTest() {
        when(memberService.getMember()).thenCallRealMethod();
        assertThat(memberService.getMember().getName()).isEqualTo("mike");
    }
}
```

* **Stuber Method**  
{Stubber Method}.when({Stubbing Class}).{Stubbing Method}

| Method | Description |  
|---|---|
|doReturn| Defines the action after calling the stubbing method.
|doThrow| Defines the exception thrown after calling the stubbing method.
|doAnswer| Defines the action customed after calling the stubbing method.
|doNothing| Defines doing nothing after calling the stubbing method.
|doCallRealMethod| Call the real method.

Stubber method is used when stubbing must be executed. Therefore, unlike OngoingStubbing method, put a class in the parameter for `when`. Then, call the method.

For example,
```java
//example 1
List list = new LinkedList();
List spy = spy(list);

//ERROR : list is empty, So, IndexOutOfBoundsException is thrown when call .get(0). Therefore it can't return "foo".
when(spy.get(0)).thenReturn("foo");
//For this reason, doReturn is used in this case.
doReturn("foo").when(spy).get(0);


//example 2
when(mock.foo()).thenThrow(new RuntimeException());

//ERROR : RuntimeException is thrown when call mock.foo() at previous line. Therefore, it cannot return "bar".
when(mock.foo()).thenReturn("bar");
//For this reason, doReturn is used in this case.
doReturn("bar").when(mock).foo();
```

**Methods whose return type is void can also be tested using stubber method.**  
`when` method require parameter T, but void method doesn't have return value.

Example of the Stubber Method
```java
public class MemberService() {
    public Member getMember() {
        return new Member("tester", "mike");
    }

    public int getLoginErrorNum() {
        return 1;
    }

    public void deleteMember() {}
}

@ExtendWith(MockitoExtension.class)
public class StubberMethod {

    @Mock
    MemberService memeberService;

    @Test
    void doReturnTest() {
        Member member = new Member("hello", "terry");
        doReturn(member).when(memberService).getMember();
        assertThat(memberService.getMember()).isEqualTo(member);       
    }

    @Test
    void doThrowTest() {
        doThrow(new RuntimeException()).when(memberService).deleteMember();
        assertThatThrownBy(() -> memberService.deleteMember()).isInstanceOf(RuntimeException.class);
    }
}
```

* **Calling different stubbing for each method call**
You can use the consecutive OngoingStubbing method call to different stubbing for each method call.

For example,
```java
@ExtendWith(MockitoExtension.class) 
public class ConsecutiveStubbing {

    @Mock
    MemberService memberService;

    @Test
    void consecutiveStubbingTest() {
        Member memeber = new Member("tester", "mike");

        when(memberService.getMember())
            .thenReturn(member);
            .thenThrow(new RuntimeException());
        
        //First call : .thenReturn(member)
        assertThat(memberService.getMember()).isEqualTo(member);

        //Second call : .thenThrow(new RuntimeException())
        assertThatThrownBy(() -> memberService.getMember()).isInstanceOf(RuntimeException.class);
    }
}
```

<br>
<br>
  
### How to verify Stubbing methods
You can verify stubbing methods to verify whether the stubbed method has been executed, whether it has been executed n tmes, or whether the execution has not exceeded.
```java
verify(T mock, VerificationMode mode)
```

Here is the table for VerificationMode  
|Method|Description|
|---|---|
|times(n)|Verificate of how many times it has been called.|
|never|Verificate that it has never been called.|
|atLeastOne|Verificate that it has been called at least once.|
|atLeast(n)|Verificate that it has been called at least 'n' times.|
|atMostOnce|Verificate that it has been called at most once.|
|atMost(n)|Verificate that it has been called at least 'n') times.|
|calls(n)|Verificate that how many times it has been called.|
|only|Verificate that only that verification method was executed.|
|timeout(n)| If verification takes more than 'n' ms, verify fail and end.|
|after(n)| Checking that verification takes more than 'n' ms.|
|description|A description that appears if verification fail|


For example,
```java
@ExtendWith(MockitoExtension.class)
public class VerifyMethod {

    @Mock
    MemberService memberService;

    @Test
    void verifyTimesTest() {
        memberService.getMember();
        memberService.getMember();

        verify(memberService, times(2)).getMember();
    }

    @Test
    void verifyNeverTest() {
        verify(memberService, never()).getMember();
    }

    @Test
    void verifyAtLeastOneTest() {
        memberService.getMember();
        
        verify(memberService, atLeastOne()).getMember();
    }

    @Test
    void verifyAtLeastTest() {
        memberService.getMember();
        memberService.getMember();
        memberService.getMember();
        //if memberService.getMember() is called only once, then test fails.
        verify(memberService, atLeast(2)).getMember();
    }

    @Test
    void verifyAtMostOnceTest() {
        memberService.getMember();
        //if memberService.getMember() is called more than twice, then test fails.
        verify(memberService, atMostOnce()).getMember();
    }

    @Test
    void verifyAtMostTest() {
        memberService.getMember();
        memberService.getMember();
        //if memberService.getMember() is called more than three times, then test fails.
        verify(memberService, atMost(2)).getMember();
    }

    @Test
    void verifyOnlyTest() {
        memberService.getMember();
        //memberService.getLoginErrNum();
        //if memberService.getLoginErrNum() is called, then test fails. (Only getMember)
        verify(memberService, only()).getMember();
    }
}
```






***reference***  
https://effortguy.tistory.com/143
