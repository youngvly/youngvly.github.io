---
title: "Exception Handling in Spring Controller"
date: 2019-01-08 22:37:00 -0400
categories: SpringBoot
---

Exception이 일어날 경우  try catch로 잡는다. 
# Error
제어할 수 없는 예외.
unChecked Exception.  

# Exception
예상할 수 있는 예외 혹은 프로그래머가 만드는 예외.
- Checked Exception
    throws, catch의 처리가 필요한 예외.
    ex) IOException , FileNotFoundException
- Unchecked Exception
    예외는 발생할 수 있지만, 처리하지 못할 수 있으니 상위 호출자에게 throw 하는 예외.
    catch하지 않아도 compile된다. 
    ex) RuntimeException - NullPointerException

1. Exception.printStackTrace() 의 취약점
    - 과도하게 많은 시스템 정보가 출력된다. 
    - 시스템의 스택 정보가 콘솔로 출력된다.
    런타임 예외인 경우 프로그램에서 예외처리를 하지 않음으로써 에러 메시지가 사용자의 브라우저로 노출된다. 
2. 예외처리 유형
    1) 예외복구
        다른 작업 흐름으로 유도 : 예외가 발생해도 App은 정상적인 진행을 유지.
        ```java
            try{

            }catch (Exception e){

            }finally{

            }
        ```
        
    2) 예외처리 회피
        처리하지 않고 호출한 쪽으로 throw
        ```java
        public void func() throws IOException {}
        ```
        **
    3) 예외 전환
        명확한 의미의 예외로 전환한 후 throw
        ```java
        catch(IOException e){
            throw SomeException();
        }
        ```
    4) 무시
        로그조차 필요없는 경우.(?)
3. 바람직한 예외처리
    - **로그를 남겨야 한다**
    : 어떤 문제가 발생했는지 로그를 보고 확인하고, 그 문제를 수정할수 있게 하기 위함, 로깅 프레임워크를 사용하는것이 좋다.
    ```java
     log.error("error occured" , e);
    ```
    - **예외는 구체적으로 잡아야한다.**
    : Exception은 최후의 수단으로. 예외마다 다른 처리를 할 수 있어야함.
    - **에러 페이지의 활용**
    40*, 50* 의 에러가 나올때 띄울 페이지를 따로 제작해두는 것이 좋음.
    - **try-catch블럭은 너무 길면 안된다**
    catch한 예외를 어디서 던졌는지 눈으로 볼 수 없는 문제가 생길 수 있음.
4. Spring Controller에서 예외처리
    - **@ExceptionHandler**
    : @ResponseStatus 없이 예외를 처리할 수 있다. -> 사용자를 특정 에러페이지로 리다이렉트 한다 / 
    ```java
    /* 1 - SQLException 이 발생하면 잡아서 NotFound로 응답한다.*/
    @ExceptionHandler(SQLException.class)
    @ResponseStatus(value=HttpStatus.NotFound, reason="SQL Exception Occured")
    public String func(){}

    /* 2 - 에러페이지 뷰로 에러내용 모델 넘기기 */
    @ExceptionHandler({SQLException.class,DataAcessException.class})
    public ModelAndView dbError(HttpServletRequest req , Exception e) {
        logger.error("Request : " + req.getRequestURL() + "raised : " + e);
        ModelAndView mav = new ModelAndView();
        mav.addObject("exception" , e);
        return mav;
        }
    ```
    - **@ControllerAdvice**
    : 전역 예외처리
    


*참고*
[1] Exception 처리 : https://www.slipp.net/questions/350
[2] 예외처리 가이드 : https://www.slideshare.net/dhrim/ss-2804901
[3] Spring 예외처리*** : http://springboot.tistory.com/25
[4] ExceptionHandler 사용 : http://hellokiseok.tistory.com/entry/ExceptionHandler-%EC%82%AC%EC%9A%A9%EB%B2%95-ContentType-%EB%B3%84-%EC%B2%98%EB%A6%AC
