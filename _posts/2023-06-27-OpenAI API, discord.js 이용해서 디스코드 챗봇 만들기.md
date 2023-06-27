---
layout: single 
title: OpenAI API, discord.js 이용한 디스코드 챗봇
toc: true
toc_label: "OpenAI API, discord.js 이용한 디스코드 챗봇"
toc_sticky: true
categories:
  - discord_bot
---



## discord.js



## 제작 목적

gpt api를 활용해서 디코에서도 gpt를 사용해보면 좋을거 같아서 제작하게 되었습니다

gpt 모델에 제 개인정보들을 프롬프트로 전달시켜서 마치 저와 같은 반응을 보여주고 싶었는데

말투, 나이, 이름등을 같이 전달하게 되면 소모되는 토큰 값이 많아져서 그냥 원래 기능 위주로 구현했습니다





## Discord.js

node.js 라이브러리인 discord.js 를 이용해서 사용자의 메세지에 반응하게 만들었습니다.

[공식 라이브러리 가이드](https://discordjs.guide/#before-you-begin)를 보고 참고했습니다.

![image-20230623152919466](\assets\images\image-20230623152919466.png)



[message.content 빈 값으로 출력될 때](https://limdongdang.github.io/message.content-%EA%B0%80-%EB%B9%88-%EA%B0%92%EC%9C%BC%EB%A1%9C-%EC%B6%9C%EB%A0%A5%EB%90%98%EB%8A%94-%EB%AC%B8%EC%A0%9C/)     
이 포스트 내용 참고해서 디스코드 메세지 이벤트를 만들 수 있습니다.



## OpenAI

OpenAI에서 제공하는 api를 이용해서 gpt의 다양한 모델들을 이용해 볼 수 있었습니다.

[API 사용 가이드](https://platform.openai.com/docs/api-reference/chat/create) 이 링크를 참고하여 gpt 3.5 turbo 모델을 이용하여 제작해봤습니다



### GPT 3.5 turbo

---

비용 면에서 더 저렴하고 강력한 기능을 제공하는 모델이라고 합니다

| **Model**   | **Input**           | **Output**         |
| ----------- | ------------------- | ------------------ |
| 4K context  | $0.0015 / 1K tokens | $0.002 / 1K tokens |
| 16K context | $0.003 / 1K tokens  | $0.004 / 1K tokens |

상당히 저렴한 가격을 볼 수 있는데

실제로 테스트겸 친구들의 디코 서버에 약 30분간 잠깐 틀어놓고 120번 정도의 api 호출을 받아봤는데 대략 0.05달러 즉 약 60원... 이 나왔습니다



### 코드 설명

---

#### role에 대해서

----

role 역할 지정을 통해서 설정을 지정합니다.

![image-20230623152919466](\assets\images\디코봇.PNG)

중요한 부분만 보자면

```js
//system 역할을 사용해서 설정값을 넣어줄 수 있음 다만 토큰을 좀 더 소모하게 됨
{role : "system", content : "너의 이름은 림동봇이고, 디스코드에서 사람들의 질문에 응답하는 봇이야"},
{role : "user", content: reqMessage.content}
```

두 가지 역할로 나뉜걸 볼 수 있는데 ```system``` 역할로 지정하고 ```content``` 에 넣은 값으로 설정을 할 수 있습니다.



![image-20230623152919466](\assets\images\디코봇1.PNG)

#### 채널에 응답 결과 출력

---

api를 호출하고 응답 결과를 채널에 보내는 코드입니다.

message 객체를 gptapi에 전달해서 함수 내부에서 바로 보내도록 구현했습니다.



**메세지 이벤트**

```js
client.on('messageCreate', (message) => {
	if (!message.author.bot) { // 봇이 보낸 메시지가 아닌 경우에만 반응
		gptapi(message);
	}
  });
```

**API 호출 이벤트**

```javascript
async function gptapi(reqMessage){
	...
    ...
	// 토큰 사용량 출력 평균 100~200
	console.log(chatCompletion.data.usage);
	// 채널에 응답 메세지 보내기
	reqMessage.channel.send(chatCompletion.data.choices[0].message);
}
```



[**전체 코드**](https://github.com/Limdongdang/Toy-Project/tree/main/gpt-discord-bot)
