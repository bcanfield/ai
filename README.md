# Vercel AI Utils

Edge-ready utilities to accelerate working with AI in JavaScript and React.

## API

### `OpenAIStream(res: Response): ReadableStream`

A transform that will extract the text from all chat and completion OpenAI models as returned as a `ReadableStream`.

```tsx
// app/api/generate/route.ts
import { Configuration, OpenAIApi } from 'openai-edge';
import { OpenAITextStream, StreamingTextResponse } from '@vercel/ai-utils';

const config = new Configuration({
  apiKey: process.env.OPENAI_API_KEY,
});
const openai = new OpenAIApi(config);

export const runtime = 'edge';

export async function POST() {
  const response = await openai.createChatCompletion({
    model: 'gpt-4',
    stream: true,
    messages: { role: 'user', content: 'What is love?' },
  });
  const stream = new OpenAITextStream(response);
  return new StreamingTextResponse(stream);
}
```

### `HuggingFaceStream(iter: AsyncGenerator<TextGenerationStreamOutput>): ReadableStream`

A transform that will extract the text from _most_ chat and completion HuggingFace models and return them as a `ReadableStream`.

```tsx
// app/api/generate/route.ts
import { HfInference } from '@huggingface/inference';
import { HuggingFaceStream, StreamingTextResponse } from '@vercel/ai-utils';

export const runtime = 'edge';

const Hf = new HfInference(process.env.HUGGINGFACE_API_KEY);

export async function POST() {
  const response = await Hf.textGenerationStream({
    model: 'OpenAssistant/oasst-sft-4-pythia-12b-epoch-3.5',
    inputs: `<|prompter|>What's the Earth total population?<|endoftext|><|assistant|>`,
    parameters: {
      max_new_tokens: 200,
      // @ts-ignore
      typical_p: 0.2, // you'll need this for OpenAssistant
      repetition_penalty: 1,
      truncate: 1000,
      return_full_text: false,
    },
  });
  const stream = new HuggingFaceStream(response);
  return new StreamingTextResponse(stream);
}
```

### `StreamingTextResponse(res: ReadableStream, init?: ResponseInit)`

This is a tiny wrapper around `Response` class that makes returning `ReadableStreams` of text a one liner. Status is automatically set to `200`, with `'Content-Type': 'text/plain; charset=utf-8'` set as `headers`.

```tsx
// app/api/generate/route.ts
import { OpenAITextStream, StreamingTextResponse } from '@vercel/ai-utils';

export const runtime = 'edge';

export async function POST() {
  const response = await openai.createChatCompletion({
    model: 'gpt-4',
    stream: true,
    messages: { role: 'user', content: 'What is love?' },
  });
  const stream = new OpenAITextStream(response);
  return new StreamingTextResponse(stream, {
    'X-RATE-LIMIT': 'lol',
  }); // => new Response(stream, { status: 200, headers: { 'Content-Type': 'text/plain; charset=utf-8', 'X-RATE-LIMIT': 'lol' }})
}
```