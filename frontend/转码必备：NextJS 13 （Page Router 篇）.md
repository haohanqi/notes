前提回顾：

上一篇我们说到了React 18之后的新功能：**server component** 和 **suspense**

但目前 React 18 却不支持这两个新功能，React team选择把它们交给NextJS 和 Remix 去完成。

于是大部分公司开始转向使用支持ssr的NextJS和Remix，面试的方向中除了基础的React知识，NextJS和Remix也变为不可缺少的部分。

本篇内容：

这一篇先从NextJS的` page router` 开始讲起

1. 首先明白 `client side render` 和 `server side render` 的区别
2. 然后理解 NextJS 想要解决什么问题 ？
3. 最后 `page router` 提供了什么功能 ？

预告：

下一篇会在本篇的基础上开始聊一聊 Next 13 中的 `server component`



## Client Side Render 和 Server Side Render 的区别
---

==想要明白 NextJS 和 React之间的区别，或者说 NextJS 解决了什么问题，那么就要先从 `Client Side Render` 和 `Server Side Render` 的区别谈起 ==

### Client Side Render

> **举个点菜的例子**

   你去知名“客户端渲染”餐馆点菜，==餐馆有个奇怪的规矩，就是前菜，凉菜，热菜，甜点一起上==。也就意味着，你只需要点一次菜，然后等待上菜，菜齐了之后你想怎么吃就是你自己的选择了。

> **在浏览器的语境下：**

   我们请求一次服务器（点菜），服务器返回我们需要的minimum html 和 js，后续的操作比如：导航别的页面，点击按钮页面产生变化...都是基于返回的js在本地进行，不需要再次请求服务器。

> **优点：**

   1. 页面的响应速度快：尤其是关于 state 变化的操作，因为所有的操作都是通过客户端的 js 进行。给人一种使用App的感觉。
   2. 渲染速度快：**在 first loading 结束之后**，页面的渲染速度会很快。

> **缺点**

   1. First loading 的时间会比较长：因为毕竟“后厨”需要一次性把所有的“菜”都上齐，所以相对的等待时间会比较长。
   2. 因为这段空白的等待时间带来两个结果：
	* 用户体验不好
	* 等待期间因为什么都没有渲染，搜索引擎对我们的页面一无所知，导致SEO变现不好

---

### Server Side Render

> **举个点菜的例子**

   你去知名“服务器渲染”餐馆点菜，餐馆有个奇怪的规矩，你点一道菜，上一道菜，然后你才能继续点餐。这也就意味着虽然上菜的速度变快了，但是你需要不断的去点菜。

> **在浏览器的语境下：**

   我们请求一次服务器（点菜），服务器返回我们需要的页面结果（full html），之后的操作比如：登陆，请求别的页面，点击按钮...都需要我们再次请求服务器，等待服务器返回结果。

> **优点：**

   1. First Loading速度快：相对于Client side render，我们只点了“一道菜”，上菜的速度势必要快。而且server side render返回的是html结果，所以客户端也不需要下载大量的 js。
   2. 更好的SEO表现：因为First Loading速度快，搜索引擎可以读取页面的内容，SEO表现会大大提升。
   3. 节省Fetch请求时间：因为所有的渲染都发生在服务器，也就意味我们可以直接在服务器拿到所需要的数据。

> **缺点：**

1. 与用户的交互感不强，因为每次渲染都需要请求服务器。
2. 面对现代网页的都太渲染需求，服务器的负担比较大。

---

## 总结两种渲染方式：

> **从两种渲染方式的优缺点不难看出：**

1. client side render的首次加载时间慢，有一个巨大的空档期，用户和搜索引擎什么都看不到，但是加载完成之后的用户操作会很快。

2. server side render的首次加载时间快，返回的html对用户和seo表现都很友好，但是加载完成之后用户的操作会慢。

---

## NextJS 想要解决什么问题 ？

NextJS想解决的问题很简单：==既然两种不同的渲染方式各有利弊，那么我们如何把两种结合起来，扬长避短，在不同的情况下使用不同的渲染机制。==

为了解决这个问题，**NextJS想出来两个方法：**

> **在两种渲染方式取中间点**

   这两种渲染方式多少都有点“极端”，那么可以采取一个中庸的方式来解决

   1. 我们先利用 `server side rende`r 在 `build time` 渲染一个 html 来弥补 client side render 首次渲染的空档期。==这种中庸的渲染方式就是：static site generation，顾名思义在首次渲染的时候生成静态html==

   2. 但问题是一个静态的 `html` 是没办法让用户操作的，那么这个时候我们再继续使用 `client side render` 在静态 `html` 的基础上继续完成渲染。==之后的 `client side render` 被称为 `hydrate`（注水），想象  js 从 html 顶部像是注水一样一直浇灌下去，让我们的静态 html 可以动起来==


> **根据页面需求选择渲染方式**

   **NextJS页面渲染把情况归为两种：**

   - 如果是一个静态页面（任何用户看到该页面都是一样的），直接使用 `static site generation`

   - 如何页面是根据用户的请求而发生变化，那么就使用 `server side render`

   **从前端开发者的角度来思考：**

   ==当我们决定一个页面在NextJS中如何渲染，我们需要问自己 “这个页面能不能在用户请求之前提前渲染这个页面“==

   - 如果可以：直接使用 `static site generation`
   - 反则：`server side render`

---
 
## Page Router 提供了什么功能

如果你理解了以上概念，那么让我们看看实际的操作

> **建立一个新的Next项目**

```
npx create-next-app@latest
```

   在新建的项目中你会发现一个 `pages` 文件夹

   这个文件夹下的所有 `tsx` 或者 `jsx` 都会被定义为 route

	   比如：`index.tsx` 就对应着 `home page`， 也就是我们说的 `/`
	   比如：`setting.tsx` 就对应着 `setting page`，也就是 `/setting`

   如果你熟悉纯react项目，我们不需要自己用 `react-router` 来配置路由

> **Page** 

   所有在 `pages` 文件夹内的 component 都可以 `export` 两个特殊的 `function`

   - getStaticProps

   - getServerSideProps

   我们之前说了，next 可以让开发者选择页面的渲染方式。

   - 如果使用 `getStaticProps` 意味着使用 `static site generation` 渲染方式

   - 如果使用 `getServerSideProps` 意味着使用 `server side render` 渲染方式

> **`getStaticProps`**

   next 会默认 `static site generation` 渲染所有的页面，既然是默认选项，那么这个 function的作用又是什么？

   - 回顾 `client side render` 请求数据
       
       在传统的 react 项目中，比较常见的一个模式就是在页面渲染完成之后，展示数据。
   
       解决方案：在 `useEffect` 请求数据
   
```jsx
const page = ()=>{

 useEffect(()=>{
   const getData = async ()=>{
	const result = await fetch(....)
	setResult(result)
   }
	getData()
 },[])

 return <>
	 ....
 </>

}
```

   - 什么是 `getStaticProps` 请求数据

      ==其实 `getStaticProps` 做的事情和在 `useEffect`中请求数据非常类似，但因为我们现在需要提前渲染页面，请求的时间点是不一样==
   
      `client side render`：当页面渲染完成，请求数据
      `static site generation`：在 build time 请求数据（当我们 `npm run next build`的时候）
   
      具体实例：
   
```jsx
// 接受从 getStaticProps 获得的数据
export default function Blog({ posts }) { 
// Render posts...
}

// 在 build time 请求 blog 数据
export async function getStaticProps() { 
	const res = await fetch('https://.../posts') 
	const posts = await res.json() 
	// 作为 Blog props 传递给 Blog component
	return { props: { posts, }, }
}
```
   
> **`getServerSideProps`**

如果理解了上面的 `getStaticProps`， `getServerSideProps` 的功能是一样的。

区别在于请求的时间点不同：`getServerSideProps` 是在服务器渲染时请求。


## 总结

以上就是 NextJS 的一些重要概念，但这里还是有很多我没有提到的地方，如果你已经可以理解以上的概念，那么你可以在NextJS官网上继续学习一下概念：

- Incremental Static Regeneration (ISR)
- getStaticProps
- Custom document

这一篇主要讲的是 Page Router 功能，其中一个问题大家可以思考一下，既然我们可以在 Page level 选择页面的渲染方式，那么可不可以具体到更深一层的 component level ？

下一篇我们一起开看看 Next 13更新之后的新功能 App Router 和 server component。

   
   