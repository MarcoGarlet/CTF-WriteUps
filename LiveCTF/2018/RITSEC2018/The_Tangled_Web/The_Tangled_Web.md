## exploit
I made this simple python program that for each html pages explored store in a list all `href` and a `true/false` value to check if a node was yet visited.
The script searched for `Flag` in a web page and printed it out, so returned `Fl4gggg1337.html` pages but without what we wanted. 
Inside `Fl4gggg1337.html` I noticed a link called `!` to `Stars.html` page. Inside that page I noticed a paragraph with this base64 string:

```
UklUU0VDe0FSM19ZMFVfRjMzNzFOR18xVF9OMFdfTVJfS1I0QjU/IX0=
```
and after decode I had the flag:

##### FLAG:RITSEC{AR3_Y0U_F3371NG_1T_N0W_MR_KR4B5?!}
