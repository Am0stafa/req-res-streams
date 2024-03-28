# req-res-streams
Long pooling aka http streaming


# Streaming OpenAI API Responses with Long Polling

The OpenAI API allows streaming responses for chat and completions. This is useful for showing incremental outputs in real-time. However, streaming requires keeping an open connection to the API. Here is one way to implement streaming with long polling in JavaScript.

## Server Side

On the server, make the request to the OpenAI API with `stream: true` and `responseType: 'stream'`. This will return a readable stream.

```js
const response = await openai.createCompletion({
  model: 'text-davinci-003', 
  prompt: 'Hello',
  stream: true,  
}, {
  responseType: 'stream'
})
```

Process the stream in chunks and send each chunk to the client:

```js
response.on('data', (chunk) => {
  const text = getTextFromChunk(chunk) 
  res.write(text) 
})
```

## Client Side 

On the client, make a request to the server endpoint. The response will be a stream of text chunks.

To implement long polling:

- On each response, check if data exists. 
- If yes, update state and re-request immediately. 
- If no data, wait 1 second and re-request.

```jsx
const [texts, setTexts] = useState([])

const requestStream = async () => {

  const res = await fetch('/endpoint')
  
  if (res.body) {
    const text = await res.text()
    setTexts(prev => [...prev, text])
    requestStream() // re-request immediately
  } else { 
    setTimeout(() => requestStream(), 1000) // retry after 1 sec
  }

}
```

This allows displaying the stream while keeping the connection open with repeated requests.
Let me know if you would like me to clarify or expand on any part!
