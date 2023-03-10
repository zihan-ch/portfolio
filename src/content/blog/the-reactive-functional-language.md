---
title: A reactive functional language
date: 04/02/2023
description: Imagining a new reactive functional language.
tags: [language, reactivity]
---

JSX/TSX is nice. But when I am using them, I found it bothering to write these things all the time:

```tsx
// react
const [count, setCount] = useState(0)
useEffect(() => console.log(count), [count])
setCount(count + 1)

// solid
const [count, setCount] = createSignal(0)
createEffect(() => console.log(count()))
setCount((count) => count + 1)

// vue
const count = ref(0)
watchEffect(() => console.log(count.value))
count.value += 1
```

I am a lazy person at all and I just hate this. There are some plugins dealing with these kind of repetive works (e.g. `$ref` in vue). Svelte also has a easier syntax like this:

```html
<script>
	let count = 0
	$: console.log(count)
	count += 1
</script>
```

This is much better than writing `[count, setCount]`. But it's not solving the problem under the hood, that JS/TS is not a "reactive" language.

I am imaging a futuristic language. I assume that the language should be a functional language. What? Did you just said functional language? Is it gonna be something like Elm or PureScript? Nope. I'. trying to add built-in reactivity primitives to the language. Also, the HTML-like (or similar to JSX/TSX) XML syntax will also be included into this language by using the built-in Component feature. Obviously, the language is going to be fully typed (and imitate TypeScript by adding `any` type to enpower the flexibility). Also, I will add the Rust-like enums into this language. There is neither `null` or `undefined` in the language. Even further, there's no `Error`, `try` or `catch` in this language. Instead of using null types or error throwing, the language uses `Option<T>` for null types and `Result<T, E>` for errors. Let's see what the code is going to look like.

# Example

Defining functions:

```ts
function sum = (a: number, b: number) => a + b
let theSum = sum(1, 2)
```

Note that there are no classes in this language. I don't wanna use OOP in this language because I figured that almost everything you can do with OOP you can do that in functional programming too. If OOP is the true answer, why is React Hooks just much better than React Class Components?

This language uses variables as immutable by default just as what Rust does. Also you can easily shadow the variables just like what you can do in Rust.

Let's move on to my favourite part of the language, which is also the most important part in my opinion - Reactivity and components.

```html
component <Counter initial:number={0} {
    state count = initial
    computed double = count * 2
    memo triple = count * 3
}>
    <button @click={() => count += 1}>{count}</button>
    <h1>Double: {double}</h1>
    <h1>Triple: {triple}</h1>
</Counter>
```

Sorry for the ugly code highlighting. Hopefully it's not stopping you from reading it.

In this code, we defined a component called `Counter`. The initial is obviously a prop which has the type of number and a default value `0`. then we passed a block into this component (I should say define instead of pass). We defined a state `count` which has the initial value as the prop defined. Then we have a computed `double` which is always double the count and a memo `triple` which is always triple the count. You might be quite familiar with the computed values and memos if you used any frontend frameworks before. In solid, we might write this:

```tsx
const Counter: Component<{ initial: number }> = (props) => {
	const [count, setCount] = createSignal(props.initial ?? 0)
	const double = createComputed(() => count() * 2)
	const triple = createMemo(() => count() * 3)

	return (
		<>
			<button onClick={() => setCount((count) => count + 1)}>
				{count()}
			</button>
			<h1>Double: {double()}</h1>
			<h1>Triple: {triple()}</h1>
		</>
	)
}
```

The code in TSX is just much more complex and annoying. How does the components in this language work differently? The language is not just aimed for client-side HTML generation, but also server-side stuff. This means that all the HTML elements are just the component primitives that you can access when you specified the code will run in the browser (or SSR). This means, when you are doing stuff on the server, you can also use the Components to implement things!

You can think of the `button` and `h1` as something that you can import. They are no more elements, but **primitive components**. In this language, we will assume all the components with a hyphen-separated case as a primitive component while all the components with a upper camel case are the components for the app logic. This is not mandatory, but I think this is a good practice. The primitive components does not return another component, but some primitive values; non-primitive components should return another component (including primitive components). Its result will be absolutely not components, but the primitive values which are generated by the primitive components; but it will never directly return some values, but compose other primitive components to do so.

Components are certainly event-driven, which is always a nice paradigm both on the server and client (I mean on the server and in the browser with DOM). On the server, a request from a client can be assumed as an event; in the browser, the `click` event with DOM can be an event. The event may motate some states. This is also similar to `EventEmitter` in node. I like vue's `@event` syntax so I just used this.

Did I just talked about the language on the server? Let's have a quick look.

```tsx
export default runServer(<HttpServer port={3000} @request={async (request) => {
    return Response.fromString('Hello world')
}} />)

// with a router
let router = <Router>
    <Route path="/" @get={(request) => Response.fromString('with router')} />
</Router>
export default run(<HttpServer port={3000} @request={routerHandler(router)} />)
```

Oh what a graceful language...

# Conclusion

What I believe is most special about this language is probably the built-in fine-grained reactivity, component-based program, event-and-state-driven components and the power of functions. When we are using a purely functional language, what we are most concerned about is how to manage reactive states and updates. Events are also hard. This language aims to solve these problems without having to use JavaScript or TypeScript (which is originally still JavaScript).

I'm still thinking a lot about reactivity, events and how an eligible language should be like. I really want to use Rust-like enums in TypeScript and I even wrote a library which implements that in TypeScript. However, I finally found it really useless to try to adding language features to JavaScript. JavaScript is generally just a bad language. I blame JavaScript's success for its bad browser compatability. JavaScript is the first successful attempt to an embedable language for the web (at least it seems that it's successful). When people realized that many JavaScript interfaces is not really well supported by many browsers (e.g. the evil IE) or they are not the same in different browsers, it's just impossible to ask every browser provider to implement JavaScrip correctly. That's how they invented something like Babel. They made modifying JavaScript code easy and more and more build tools come to the world. With more and more build tools like Webpack at first, and then Vite, JavaScript has the best DX among all the languages. Just look at how people invented TypeScript, how react's devs created JSX/TSX, how people are trying to write HTML + JS + CSS in one single file using JSX + CSS in JS. Also how they invented fullstack web app frameworks like Next, how tRPC made fullstack typesafety easy. JavaScript succeeded not because JavaScript is a good language. It's a language built for the web. It's the language that developers use the most. They use JavaScript to build the web, they also write things about JavaScript on the internet much more than backend langauges like C. They spent countless money on building things like V8, JavaScriptCore or SpiderMonkey. They got thousands of sponsors on Github just for maintaining a frontend framework because developers need to use them. I really love TypeScript, but I'm wishing I could just get rid of the JavaScript hell.

Thank you.

[Discuss with us](https://github.com/zihan-ch/portfolio/discussions/1)
