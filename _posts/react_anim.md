# React 中的 Canvas 动画

移动端硬件性能越来越好的今天，页面的交互也越来越丰富，Web 体验也在不断向原生应用靠拢，加入了越来越多的手势与动画。除了常见的 CSS 动画外，有时候我们还会使用到 Canvas 或者 SVG 进行动画内容表现。

由于 React 在平日的开发中依旧拥有不少使用者，分享一个在 React 开发中使用 Canvas 动画的方法及其性能优化。

## 一、动画的基本原理

人的眼睛对图像有短暂的记忆效应，当眼睛看到多张图片连续快速的切换时，就会被认为是一段连续播放的动画了，而一秒内切换多少张便是所说的帧率(FPS)，它也常被用作动画流畅度的重要指标。虽然帧率是越高越好，但一般达到30帧后，便基本可以觉得流畅的。

日本动漫的手绘(EVA、进击的巨人等)、粘土动画或者 3D 渲染等不同创作方式都能制作动画，但原理都是一样的。

## 二、Web 中的动画

当聊到 Web 的动画时，我们的第一反应可能是 CSS，通过 CSS 来实现各种各样的效果——位移、旋转、透明等等。其实，除了这些特效以外，Web 也有很多其他的动画手段，其中比较主要的载体便是 ***SVG***、***Canvas***、***WebGL***。通过这些载体除了可以实现上述 CSS 的效果以外，还可以实现更复杂的内容（比如游戏动画）。

由于有些动画较为细腻且复杂，无法通过简单的位移或变形来实现(例如人物的行走、跳跃)，我们便会使用到帧动画。“帧动画”是一种常见的动画形式，是将某时间轴拆分成若干个连续的关键帧，并在的每一帧上分解动画动作、绘制不同内容，使之连续播放变为动画。帧动画也被称为“序列帧动画”、“定格动画”、“逐帧动画”。

归结起来，也就是下面这张图片要表达的意思。

![flip animation](./anim.gif)

### 实现帧动画的手段

实现帧动画的手段也有很多种，比较常用的有下面三种：

#### GIF 图片

- 优点: 成本较低、使用方便
- 缺点: GIF动画支持颜色少(最大256色)、Alpha 透明度支持差，图像锯齿毛边比较严重，灵活性较差

#### CSS

- 优点: 使用方便，支持所有图片种类（可以通过`background-position`, `transform` 属性以切换图片的方式实现），`translate3d` 属性也能充分使用 GPU
- 缺点: 实现较为复杂，多个动画间的同步可能会产生问题

#### JavaScript

利用 JavaScript 将内容绘制到 ***Canvas*** 等载体上，并通过实时计算来决定绘制的图像、位置、变形、透明度等等，也是本篇的主要介绍方法。

- 优点: 支持所有图片类型，可以实现复杂的动画控制
- 缺点: 实现较为复杂，需要使用到较多的 CPU 运算

当然，本篇还是着重介绍使用 JavaScript 的方式实现动画，进而迁移到 React。

## 三、使用 JavaScript 实现动画

如果计划使用 JavaScript 来进行动画的渲染，基本上我们都会选用一个渲染框架来将动画内容渲染，来简化我们的渲染操作、提高编码效率，当然你也可以直接使用原生 API 来进行操作，不过这样的话本身会比较烦琐。JavaScript 的渲染框架有很多，包括 ***Lottie***， ***Pixijs***， ***Threejs***， ***Createjs***， ***Konva*** 等等。这些框架本身都很成熟，而且也有针对不同的场景，我们可以根据业务特点来进行选择。因为本文并不牵涉复杂场景，因此这里选用比较简单的 ***Konva*** 来作为示例进行讲解，如果读者有兴趣的话也可以去了解一下其他的框架。

下面我们通过一些代码片段来看下如何从一个基本的 Canvas 动画，逐步的迁移到 React 中，并融合进 react-dom 中。使用 Canvas 来实现动画的实现并不复杂，可以简单地用 4 个字来概括： **定时重绘**。

### 定时

我们先来看下 **定时**，JavaScript 中的可以实现定时手段有好几种，优先级排序上：`requestAnimationFrame` > `setTimeout` > `setInterval`，因此我们首选 `requestAnimationFrame` API。原因主要是在执行优先级上，这部分内容超出本文范围，有兴趣或者不太了解的读者可以自行查阅。

通过定时任务，就可以实现动画中最基本的“tick 机制”了。

```javascript
function tick() {
    // 绘制动画内容至载体上
    // 下一帧继续执行，则调用
    requestAnimationFrame(tick);
};

// 开始执行 tick 逻辑，用于动画的不间断绘制
tick();
```

### JavaScript 位移动画

下面使用 ***Konva*** 实现一个简单矩形的位移动画，当 x 轴的移动到 30 时就停止，代码在每次定时任务触发时会重新计算矩形的位置，然后对内容进行了重新绘制。***Konva*** 对 Canvas 进行了简单的封装，将绘制内容通过对象进行管理，每次绘制前会自动进行清除操作。

```javascript
const stage = new Konva.Stage({
    container: 'container',
    width: 100,
    height: 100,
});

// 这里 Layer 是实际的 Canvas 实例
const layer = new Konva.Layer();
stage.add(layer);
let x = 0;

// 创建一个矩形并添加到 Canvas 中
const rect = new Konva.Rect({
    x: x,
    y: 20,
    width: 50,
    height: 50,
    fill: 'red',
});
layer.add(rect);

// 定时更新 rect 这个对象的位置
function tick() {
    x += 1;

    // 更新位置后便重绘
    rect.setAttr('x', x);
    rect.draw();

    if (x > 30) {
        return;
    }
    requestAnimationFrame(tick);
};

tick();
```

上面的代码，通过最简单原始的方式实现了一段 Canvas 的位移动画。由于我们平时多用 React 进行页面的渲染，因此我们希望尽量避免直接使用 JavaScript 直接操作 DOM 元素构建动画的容器或内容，更希望把它移植到 React 中。

### React 构建 div 容器

react-dom 本身允许我们绘制各种各样的 HTML 节点，因此我们利用 React 来创建画布的 `div` 容器，然后用上面相同的代码逻辑来绘制 Canvas 中的动画即可。将上面的代码稍作修改就可以移植到 React 中了，**Konva** 的 `Layer` 对象才是真正的 `canvas` 画布，所以代码中 `render` 方法返回的是 `div` 而非 `canvas`（如果你选用的框架是使用 `canvas` 元素来进行初始化的，这里也可以返回 `canvas`，依据场景决定）。

```javascript
function createPic(canvasContainer) {
    const width = 100;
    const height = 100;
    // 根据传入的 canvasContainer 来创建画布
    const stage = new Konva.Stage({
        container: canvasContainer,
        width: width,
        height: height,
    });
    // 其余与上面的代码类同
    return stage;
}

// 使用 React 函数组件方式建立 Canvas 或者对应的容器
function DrawCanvas() {
    const ref = useRef();

    useEffect(() => {
        // 将 div 容器传入方法创建对应的场景元素
        const stage = createPic(ref.current);
        // 销毁容器
        return () => {
            stage.destroy();
        }
    }, []);
    return <div ref={ref} id="canvasContainer" />;
}
```

通过上面的代码我们已经让动画的代码和 React 结合起来了，不过由于 react-dom 本身并不支持渲染 **Konva** 中的绘制元素，因此依旧有 2 种风格的代码存在，一种是 JSX 风格，另一种则是传统风格，即通过对象的添加与删除来进行管理。

接下来我们会思考另一个问题——是否能够将两种代码风格合并为一个？毕竟不同代码风格维护起来很难受（简直逼死强迫症），而且 JSX 会更加直观，更符合现在的编码习惯。所以剩下的问题就是如何将 Konva 中的 `Stage`、`Layer`、`Rect` 这些对象也通过 JSX 进行管理。

### react-konva

Konva 有提供 React 版本——react-konva，因此我们把上面的代码改写下。

```javascript
import React, { useEffect, useRef, useState } from 'react';
import { Layer, Rect, Stage, Text } from 'react-konva';
import Konva from 'konva';

const Picture = () => {
    // 这里只是为了表明这里 div 和 konva 的 Rect 能同时被绘制，因此加了一层 div 元素
    // 实际可以不需要
    return (
        <div>
            <Stage width={100} height={100}>
                <Layer>
                    <Rect
                        x={0}
                        y={20}
                        width={50}
                        height={50}
                        fill="red"
                    />
                </Layer>
            </Stage>
        </div>
    );
};
```

看到这里也许你会有一个疑问，为什么 `Layer`、`Rect` 这些 Konva 中的对象能被正确解析并绘制到页面上，react-dom 不是仅能够渲染 HTML 组件吗？

### react-konva 源码解读

react-konva 的确封装了一点内容，它实现一个自定义的 Render 来对 JSX 中的这些节点进行解析，最后将节点渲染至 Canvas 中。接下来我们抽取部分 react-konva 来分析下具体的实现(了解 React 自定义Render的可以跳过这一段)。

首先从系统上来考虑，使用自定义的 Render 来绘制这些图形节点必须要同时支持 react-dom 已有的功能，因为除了图形节点以外，系统依旧还是需要支持普通的 HTML 元素的现实的，因此 react-konva 选择基于 ***react-reconciler*** 来实现。

***react-reconciler*** 定义了各种操作接口，需要使用方来完成实现，包括创建、更新、移除等一系列操作来控制节点。react-konva 利用这套机制，将 React Element 对象转化为了 Konva 中的对象，进行内容的绘制。由于 react-konva 并不打算也不需要负责 react-dom 已有的功能，因此它在代码中将自己标示为辅助 Render，这样就不会影响到 react-dom 的渲染。react-dom 并不会主动同步多个 Render 之间的生命周期，因此我们需要通过在节点的各个生命周期中主动调用来同步 2 个 Render 的生命周期。

不过官方文档上指出 react-reconciler 相对于其他框架来说本身依旧还不稳定，API依旧会有所变动，使用起来要记得 **锁定版本号**。

> Its API is not as stable as that of React, React Native, or React DOM, and does not follow the common versioning scheme.
> Use it at your own risk.

#### Render 间的生命周期同步

下面是通过函数组件(Function Component)实现的自定义 render 与 react-dom 之间的生命周期同步的部分代码。

```javascript
import ReactFiberReconciler from 'react-reconciler';
// 这里的 HostConfig 是接口的具体实现
import * as HostConfig from './ReactKonvaHostConfig';
// 创建自定义的 render
const KonvaRenderer = ReactFiberReconciler(HostConfig);

// Stage 的函数组件
function StageWrap(props) {
  // ...do something
  const container = useRef();
  
  React.useLayoutEffect(() => {
    // 创建渲染的根节点，传入的属性略过
    // 这里使用 StageWrap 里返回的 div 作为 Stage 的容器
    // 相当于在 react-dom 中开启了第二个 render
    const stage = new Konva.Stage({ container: container.current });
    // 利用自定义创建
    fiberRef.current = KonvaRenderer.createContainer(stage);
    KonvaRenderer.updateContainer(props.children, fiberRef.current);

    // unmount 的时候对 Stage 画布本身进行销毁
    return () => {
      KonvaRenderer.updateContainer(null, fiberRef.current, null);
      stage.destroy();
    };
  }, []);
  
  // 每次 StageWrap 被触发 componentDidUpdate 时，同时更新内容
  React.useLayoutEffect(() => {
    applyNodeProps(stage.current, props, oldProps);
    KonvaRenderer.updateContainer(props.children, fiberRef.current, null);
  });
  
  return (
    <div
      ref={container}
    />
  );
}
```

#### ReactKonvaHostConfig 操作接口实现及属性设置

react-reconciler 定义了一系列预定义的接口（即我们上面引用的 HostConfig），用于处理各种场景下对于渲染对象的处理。实现自定义的绘制框架便要对这些预定义的接口进行实现，不过并非所有的方法都必须要有完整的实现，你可以根据自己的需求实现部分功能，所有接口和配置的定义详见文档。下面列出几个比较主要的定义，通过这些定义来看下如何将 React 中的节点转换为 Canvas 中实际绘制的内容的。

`createInstance`: 用于创建显示的实际节点对象，例如 `div`、`span` 等，React 的文本节点不会被传递到这里来，下面看下部分 react-konva 的 `HostConfig` 实现逻辑。

```javascript
function createInstance(type, props, rootContainer, hostContext, internalHandle) {
    // 用于创建node的节点，!!!但不可操作本节点以外的内容，包括添加删除，事件也可以在后续再添加
    
    // 这里的type是string，因此可以直接根据type来选择对应的konva对象
    let NodeClass = Konva[type];
    
    // 初始化节点的属性，由于事件不在这个方法内添加，因此从props中滤除
    const propsWithoutEvents = excludeEvts(props);

    // 创建渲染用的对象并返回
    const instance = new NodeClass(propsWithoutEvents);
    return instance;
}
```

`createTextInstance`: 用于创建文本节点 (例如 `<Text>foo</Text>`)，由于文本节点不支持属性，因此如果你不打算支持这里直接抛出异常(`throw error`)就好。

```javascript
function createTextInstance(text, rootContainer, hostContext, internalHandle) {
  console.error(
    `Text components are not supported for now in ReactKonva. Your text is: "${text}"`
  );
}
```

`appendInitialChild`、`appendChild`、`appendChildToContainer`、`insertBefore`、`insertInContainerBefore`、`removeChild`、`removeChildFromContainer`: 对 `createInstance` 中创建出来的对象进行 增/删/改 操作，以 `appendInitialChild` 举例。

```javascript
function appendInitialChild(parentInstance, child) {
  // 这边做了一个额外的判断，如果是字符串类型的子节点，则不支持
  //  (eg <Text>foo</Text>)
  if (typeof child === 'string') {
    console.error(
      `Do not use plain text as child of Konva.Node. You are using text: ${child}`
    );
    return;
  }

  // 节点添加
  parentInstance.add(child);
}
```

`commitUpdate`: 当 React 的属性更新以后，这个方法便会被调用用于更新渲染对象中的属性。

```javascript
function commitUpdate(instance, updatePayload, type, prevProps, nextProps, internalHandle) {
    // 对比新旧属性，并赋值属性到渲染对象上
    applyNodeProps(instance, newProps, oldProps);
}
```

`isPrimaryRenderer`: 是否将自己作为主 Render，这里设置为 `false`，便可以使自己作为辅助 Render。

```javascript
const isPrimaryRenderer = false;
```

#### React 的位移动画

通过上面自定义的 Render 我们已经能够将图形绘制到画布上了，最后我们把定时更新部分加上就可以了，这样便完成我们的动画了。由于是在组件内部开始的定时器，因此要记得中断。

```javascript
const Picture = () => {
    const updateRef = useRef();
    const xRef = useRef(0);
    const [x, setX] = useState(0);
    updateRef.current = setX;
    
    // 创建 tick 函数，进行动态更新，只需要执行一次就可以了
    useEffect(() => {
        let id;
        const tick = () => {
            // 更新矩形的位置
            xRef.current += 1;
            updateRef.current(xRef.current);
            if (xRef.current > 30) {
                return;
            }
            id = requestAnimationFrame(tick);
        };

        tick();
        return () => {
            cancelAnimationFrame(id);
        };
    }, []);
    
    return (
        <div>
            <Stage width={100} height={100}>
                <Layer>
                    <Rect
                        x={x}
                        y={20}
                        width={50}
                        height={50}
                        fill="red"
                    />
                </Layer>
            </Stage>
        </div>
    );
};
```

## 四、优化

### 问题

敏感的同学们应该已经注意到了，定时器每次在执行时，都会不断通过 `setState` 来进行属性的变更，这样势必会导致性能上较大的损耗，导致我们的动画看起来很卡。我们可以看下下面这张图，每次更新操作都会导致一系列的方法调用，整体的消耗很大。

![bad practice flame chart](./react_update_flame.png)

因此为了避免这个问题，我们需要对整体的实现进行重新思考。

### 渲染优化

我们在 Web 页面上会选择使用 React 来进行绘制时，一般都属于 HTML 部分与 Canvas 互动较多，或者动画本身并不复杂，虽然每一帧的内容都需要重新对元素属性进行计算，但其实需要引起树结构变化的次数并不多，因此每次更新都引发React的更新调用，就引起了很多不必要性能的消耗。为了性能的提升，我们希望尽量避免这些更新操作，节点上的属性变化直接进行修改，而不是通过 `state` 或者 `prop` 来进行控制，只在需要在对象变更的时候进行树的变更操作就可以了。

依照这个思路，我们把整体的系统重新分析，根据系统特性尝试将操作分为两部分，一部分是针对树结构(相对稳定)，用于对节点进行维护与更新(JSX)一部分则是针对绘制对象中的状态进行实时计算与绘制。我们对下面的代码进行调整。

```javascript
updateRef.current(xRef.current);
```

这块通过 state 形式进行更新的代码调整为直接更新，完后成直接渲染。

```javascript
// 对 rect 的节点，直接更新并绘制
rectRef.current.setAttr('x', xRef.current);
updatePicture(rectRef.current);
```

逻辑调整的部分不多，只是将 `state` 中存储的属性改为 `ref` 来进行存储，这样我们已经可以减少掉很多多余的操作了，我们拿上面的图与下面这张来对比下就很明显了。

![good practice flame chart](./konva_update_flame.png)

下面是改造后的完整代码

```javascript
const Picture = () => {
    // 获取 rect 的节点
    const rectRef = useRef();
    const xRef = useRef(0);
    
    // 创建 tick 函数，进行动态更新，只需要执行一次就可以了
    useEffect(() => {
        let id;
        const tick = () => {
            xRef.current += 1;
            // 直接更新并绘制
            rectRef.current.setAttr('x', xRef.current);
            updatePicture(rectRef.current);
            if (xRef.current > 30) {
                return;
            }
            id = requestAnimationFrame(tick);
        };

        tick();
        return () => {
            cancelAnimationFrame(id);
        };
    }, []);
    
    return (
         <div>
            <Stage width={100} height={100}>
                <Layer>
                    <Text text="Hello world" />
                    <Rect
                        x={x}
                        y={20}
                        width={50}
                        height={50}
                        fill="red"
                    />
                </Layer>
            </Stage>
        </div>
    );
};
```

当然上面提供的仅仅是一种优化方式，实现比较简单，可以考虑在节点间没有依赖或者优先级的场景下使用。当然还有另一种方式也可以，例如通过实现特定的接口(Interface)，通过直接来调用对象的特定方法来绕过 React 的更新机制。方法的选择完全取决于使用的场景。

## 结语

React提供了非常便捷的手段用来对渲染部分进行自定义，使用这种自定义 Render 的方式就可以让我们自己来实现一套基于 React 的渲染引擎，无论是基于 react-dom 的基础上做为 Canvas 的补充也罢，或者像react-native 一样完全实现另一个全新平台也好，都有一套相对完整的手段。

使用 React 机制给我们带来了代码统一以及数据维护的便捷。不过如果打算使用这套机制直接来做动画的话，可能会面临性能问题。因此在使用上需要依据不同的场景选择合适的优化方案。对于通常的使用场景，我们仅仅只需要尝试避免通过 `prop` 或者 `state` 来进行属性上的更新就能避免性能上无谓的开销。

如果结构频繁变化或者复杂度非常高的话，也可以考虑完全剥离两套渲染体系，根据不同的场景选择合适的方案即可。
