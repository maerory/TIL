## JavaScript For Web! 						(2018/7/3)

#### We use JS5 syntax 

Js adds action and movement to our view pages!

1. 개발자 모드에 가서 Console 창을 통해 현재 페이지의 CSS 요소에 접근할 수 있다!

   ``document.getElementsById("some element");`` 의 구문으로 document내 해당 ID를 가지고 있는 아이템을 찾을 수 있다.

   이와 비슷하게,

- `getElementsByClassName()` : ClassName으로 전부 반환
- `querySelector()` : ".btn" 같은 요소를 넣어서 btn을 포함하는 첫번째 CSS 한줄 전체를 리턴해줌
- `querySelectorAll()` : 해당 요소를 포함하는 태그를 전부 다 반환해준다

2. Assign the HTML Collections to a variable in javascript

   `var btn = document.getElementsByClassName("btn")` : put all the HTML elements to the variable _btn_

   `.setAttribute("attribute", "Changed")` : you can change the attributes of an web object with this function

3. Three useful type of console

   - `console.log `: just printing 
   - `console.dir` : finding attributes attached to that web object
   - `console.error` : find the error related

4. "Event" - finding the element: event conditions on certain action of a user

   - `elements.onclick("action")` : condition some action when clicked
   - `btn[0].onmouseover = function() { alert("SUP!")};` : conditions this function when we put our mouse over the first button (We can also use objects like 'alert', 'confirm', 'prompt' which is default to the webpage)
   - `.addEventListener(condition, action function)` : EventListener allows user to specify the condition on which action should occur 

5. Creating Functions

   ``` javascript
   var func1 = function {
       alert("Some Action")
       //함수 표현식 - You can use AFTER it is defined
   }
   function func1() {
       alert("Some Action");
       //함수 선언식 - You can use BEFORE it tis defined
   }
   //익명함수! - You create function as you go for a SINGLE-TIME-USE
   ```

   - If () is put after a function, it is always executed,  so when passing functions as variable you have to put it without parenthesis.