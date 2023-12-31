# Strict-Transport-Security header

- HTTP Security Header를 적용한 PR을 만들었는데, 셀의 한 개발자 분께서 아주 좋은 질문을 주셨습니다. 'Strict-Transport-Security가 하는 HTTPS로의 전환은 AWS에서 해주고 있는데 이 헤더를 세팅해야하는 이유가 뭐냐?' 라는 질문이었습니다. 아차 싶어서 이것에 대해 좀 조사를 해보았습니다.

<br />

---

<br />

- Strict-Transport-Security 헤더는 해당 사이트가 HTTPS를 통해서만 접근되어야 하며 향후 HTTP를 사용하여 사이트에 접근하려는 모든 시도는 자동으로 HTTPS로 변환되어야 함을 브라우저에 알리는데 사용됩니다.
- The Strict-Transport-Security header is used to inform the browser that the site should only be accessed via HTTPS, and any attempts to access the site using HTTP in the future should automatically be converted to HTTPS.

<br />

- 즉, 사용자가 특정 사이트에 HTTP를 사용하여 요청을 보내려 할 때 브라우저가 선제적으로 HTTPS로 요청하게끔 리다이렉트 시키는 변환 작업을 해달라고 지정 및 요청하는 것을 말합니다.
- This means that when a user tries to send a request to a specific site using HTTP, the browser is instructed to preemptively redirect the request to HTTPS.

<br />

- 클라이언트가 한 페이지에 처음으로 접근하고 HTTP로 요청을 보냈을 때, 서버가 HTTPS로 301 Moved Permanently 상태 코드로 리다이렉트 시킵니다. (이 때의 리다이렉트는 보통 AWS가 해주는 것이죠.) 이때 Strict-Transport-Security 헤더에 값이 함께 넘어오게 되고, 이 값을 브라우저가 기억하게 됩니다. 이렇게 되면 같은 클라이언트가 또 다시 해당 서비스에 HTTP로 요청을 보낼 경우, 이번에는 서버가 아니라 브라우저단에서 HTTPS로 요청을 보내게끔 307 Internal Redirect로 리다이렉트 시킵니다. 즉, 두 번째 접근부터는 HTTPS를 사용하게끔 강제한다는 것이죠. 아래 그림이 이를 잘 표현합니다.
- When a client accesses a page for the first time and sends a request via HTTP, the server redirects to HTTPS with a 301 Moved Permanently status code. (This redirection is usually handled by services like AWS.) At this point, the Strict-Transport-Security header is passed along, and the browser remembers this value. Consequently, if the same client tries to send a request to the service again using HTTP, this time the browser, not the server, will redirect the request to HTTPS with a 307 Internal Redirect. In essence, from the second access onward, the use of HTTPS is enforced. The image below illustrates this process well.

<br />

<img src="https://github.com/muilyang12/what_i_studied/assets/78548830/95b1d67c-65ff-453d-9955-a93792e00f8b" width=750 />

<br />
<br />

- 이 시나리오를 잘 보면 첫 번째 접속 시에는 HTTP 요청을 막을 수 없고 두 번째 요청에서 부터만 막을 수 있다는 한계가 있습니다. 그렇기에 첫 번째 접근에의 대응을 위하여 AWS 혹은 별도의 다른 방법을 통해 서버가 HTTPS 요청으로 리다이렉트 하도록 하는 처리가 필요합니다.
- Observing this scenario, it is clear that it is not possible to block HTTP requests the first time they are made; the limitation is that it can only be blocked from the second request onwards. Therefore, to address the first access, it is necessary to have a process in place through AWS or another method to redirect the server to HTTPS requests.

<br />

---

- Strict-Transport-Security 헤더를 세팅한 사이트인 네이버를 http://www.naver.com URL을 사용하여 의도적으로 HTTP로 접근하며 상태코드를 살펴보려 합니다. 결과적으로는 위의 그림에서 묘사한 순서와 거의 똑같은 순서로 상태코드가 나오게 됩니다.
- I intend to intentionally access the site Naver, which has set the Strict-Transport-Security header, using the URL http://www.naver.com with HTTP to examine the status codes. As a result, the sequence of status codes will appear almost the same as the sequence depicted in the above figure.

<br />

- 첫 번째로 HTTP를 명시한 네이버의 URL (http://www.naver.com) 에 접근 시 아래와 같이 302 Moved Temporarily 가 나오며 HTTPS 요청으로 강제하게 됩니다. (두 번째 www.naver.com이 HTTPS 요청입니다.)
- When you first access Naver's URL specified with HTTP (http://www.naver.com), you get a 302 Moved Temporarily response, which forces the request to HTTPS (the second www.naver.com is the HTTPS request).

<br />

<img src="https://github.com/muilyang12/what_i_studied/assets/78548830/1d0a8e71-01e5-4960-b4ba-b87d47b92575" width=750 />

<br />
<br />

- 그 후 같은 브라우저로 다시 HTTP를 명시한 네이버의 URL (http://www.naver.com) 에 접근하면 아래와 같이 307 Internal Redirect 상태코드를 받으며 HTTPS 요청으로 리다이렉트 됩니다. 위의 순서도와 일치하죠.
- After that, if you access Naver's URL specified with HTTP (http://www.naver.com) again using the same browser, you will receive a 307 Internal Redirect status code, and the request will be redirected to HTTPS. This matches the sequence described above.

<br />

<img src="https://github.com/muilyang12/what_i_studied/assets/78548830/0e672d02-94a2-447f-bfd3-5eed08fe4be0" width=750 />

<br />
<br />

- 이와 달리 Strict-Transport-Security 헤더가 설정되지 않은 사이트에 HTTP 요청으로 접근하면 여러 번 반복 접근하여도 항상 301 Moved Permanently 상태코드가 나오며 리다이렉트 됩니다. 물론 캐시가 되기에 “307 Moved Permanently (from disk cache)” 상태 코드가 나오고 있습니다. (어찌 보면 이것도 브라우저 단에서 막은 거라고 볼 수도 있기는 합니다.)
- Conversely, if you access a site that has not set the Strict-Transport-Security header with an HTTP request, you will always receive a 301 Moved Permanently status code with repeated accesses, redirecting you each time. Of course, because it is cached, you might see the status code "307 Moved Permanently (from disk cache)." (In a way, this can also be seen as being blocked by the browser.)

<br />

- 출처
  - [What Is HSTS and Why Should I Use It? | Acunetix](https://www.acunetix.com/blog/articles/what-is-hsts-why-use-it/)
