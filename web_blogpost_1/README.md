# Web BlogPost

![Blog Post Challenge](https://github.com/notsuspiciousindividual/Stack_The_Flag_CTF_22/blob/main/web_blogpost_1/Challenge.PNG)

## Summary

Blogpost is a easy web challenge that one of the challenges that took most of my time in the CTF. Finding the foothold was an interesting process but this challenge has reminded me not to create a tunnel vision when it comes to hacking and ensuring that I check all files to have a good understanding what the challenge is.

## Setting up the environment

![When unzipped, there should be the docker environments to run](https://github.com/notsuspiciousindividual/Stack_The_Flag_CTF_22/blob/main/web_blogpost_1/settingupchallenge.PNG)

The challenge developers were kind enough to create the environment for individuals to run on their local machine. To create the environment itself, you should have docker installed in your machine. Just a little google can help you (depending on your machine) but here was how I installed it; [Link](https://www.kali.org/docs/containers/installing-docker-on-kali/)

## Enumeration

Playing around with the website and looking at the sourcecode, we have a rough understand of what the website has and the general flow for normal users; 

```
Goes to Homepage
Login
	If no user in website
		make user
	Go to Blog, server takes content from blog
	make own post etc
```

Going through the sourcecodes itself, we also understand that there'll be a bot that actively crawls through the blogpost with the required cookie we have to steal in bot.js

```javascript
export const viewPosts = async () => {
    try {
		const browser = await puppeteer.launch(browser_options);
		let context = await browser.createIncognitoBrowserContext();
		let page = await context.newPage();

		let token = await sign({ username: 'admin' });
		await page.setCookie({
			name: "session",
			'value': token,
			domain: "127.0.0.1",
		});
		await page.setCookie({
			name: "flag",
			'value': "REDACTED",
			domain: "127.0.0.1",
		});
		await page.goto('http://127.0.0.1:1337/blog', {
			waitUntil: 'networkidle2',
			timeout: 8000
		});
		await browser.close();
    } catch(e) {
        console.log(e);
    }
};
````

## Getting the foothold

![Form has no sanitisation](formsjsnosanitisation.PNG)


Looking through the sourcecode, there is no form of sanitization when it comes to posting. With that in mind, let's try it with a simple XSS. If it works, there'll be our foothole for this challenge.

```javascript
<script>console.log</script>
```

![XSS FootHold](XSSFootHold.PNG)

And it does work! Now withll our foothold, what can we do with it?


## Exploitation

