---
created: 2024-06-23 15:23
modified: 2024-06-23 15:23
Type: Entity
tags:
  - 编程/React
  - 编程/FE/Three
publish: true
date: 2024-08-11
Demo 地址: https://cube.songxingguo.com/
语雀地址: https://www.yuque.com/songxingguo/read/vkqpzpzrhwcceyh0
path: react-three-fiber-cube
categories:
  - 前端
description: 之前一直对Web3D比较感兴趣，学了Three.js，但一直没有练手机会，学习了不用的话很快就会忘，于是想着之前有玩过魔方，想着将魔方做出网页版本的，最后做成一个在线学习魔方的网站，还希望希望能通过图像识别魔方，自动生成解法。
title: 用React-three-fiber实现网页版魔方
Obsidian地址: obsidian://open?vault=content&file=C%20Knowledge%2F%E5%89%8D%E7%AB%AF%2F%E5%BC%80%E5%8F%91%E6%8A%80%E6%9C%AF%2FReact%2F%E7%94%A8React-three-fiber%E5%AE%9E%E7%8E%B0%E7%BD%91%E9%A1%B5%E7%89%88%E9%AD%94%E6%96%B9.md
---
## 目录
## 写在前面

之前一直对Web3D比较感兴趣，学了Three.js，但一直没有练手机会，学习了不用的话很快就会忘，于是想着之前有玩过魔方，想着将魔方做出网页版本的，最后做成一个在线学习魔方的网站，还希望希望能通过图像识别魔方，自动生成解法。

这次技术选型没有使用原生 Three.js 实现，之前写过一篇[快速上手DEMO](https://brain.songxingguo.com/content/dv/three/getting_started/ThreeDemo.html)，大家可以看到使用原生Three.js，即使什么也不做只是在场景中放一个矩形就接近50行代码了，确实有些麻烦。这次选择 [react-three-fiber](https://fiber.framer.wiki/)， 这是一个使用React对Three.js进行封装的3D 渲染库，为了将我新学的技术全部都使用上了，这次的技术选型采用  React + Vite +TypeScript+ React-three-fiber的方式实现。

魔方效果可以通过扫描下方的二维码或者访问 [https://cube.songxingguo.com/](https://cube.songxingguo.com/)进行查看，源码可以在 [Github](https://github.com/songxingguo/demo/tree/main/rubik-cube ) 中查看。
![ThreeJS实现网页版魔方 20240808085447473](https://image.songxingguo.com/obsidian/20240809/ThreeJS%E5%AE%9E%E7%8E%B0%E7%BD%91%E9%A1%B5%E7%89%88%E9%AD%94%E6%96%B9-20240808085447473.webp)
当然这次，还只是一个初步的模型，还有很多问题，但后面会不断迭代，添加新的功能，比如切换阶数、打乱、还原、以及教程等，并且希望能将我新学的技术都能用到上面。

## 实现思路

这次是魔方的简单实现，一共分为两步，第一步是实现一个静态魔方，第二步就是让它可以转起来。

## 实现代码

### 入门DEMO

在正式开始前，我们先用 [react-three-fiber](https://fiber.framer.wiki/) 实现一个最简单的DEMO，同样是实现 [快速上手DEMO](https://brain.songxingguo.com/content/dv/three/getting_started/ThreeDemo.html) 中的将一个立方体渲染出来功能，只需要在画布中添加 `<Canvas>`  ，也就相当于three.js中的场景（scene），然后就可以在场景中添加灯光、几何体以及组合这些物体的[网格 (Mesh)](https://link.juejin.cn?target=https%3A%2F%2Fthreejs.org%2Fdocs%2F%23api%2Fzh%2Fobjects%2FMesh) 。

```jsx
import "./App.css";
import { Canvas } from "@react-three/fiber";

function App() {
  return (
    <>
      <Canvas>
        <ambientLight intensity={0.1} />
        <directionalLight color="red" position={[0, 0, 5]} />
        <mesh>
          <boxGeometry />
          <meshStandardMaterial />
        </mesh>
      </Canvas>
    </>
  );
}

export default App;
```

可以看到，这里只使用了十几行代码，并且只需要按照语法进行配置即可，一目了然，是不是比使用纯原生的Three.js简单了许多。

### 静态魔方

#### 生成方块

下面我们正式开始实现一个魔方。首先，我们要根据魔方的阶数生成多个小方块，并将它们按一定的位置组装在一起。以最简单的二阶魔方为例，它一共有八个小方块组成，那我们就用循环遍历生成八个小方块，并且计算出它们各自的位置。

```jsx
  const genCubes = (
    x: number,
    y: number,
    z: number,
    num: number,
    len: number
  ) => {
    //魔方左上角坐标
    const leftUpX = x - (num / 2) * len;
    const leftUpY = y + (num / 2) * len;
    const leftUpZ = z + (num / 2) * len;

    const materials = genMaterials();
    const opacityRubik = getOpacityRubik();

    const cubes = [];
    cubes.push(opacityRubik);

    for (let i = 0; i < num; i++) {
      for (let j = 0; j < num * num; j++) {
        let position = { x: 0, y: 0, z: 0 };
        //依次计算各个小方块中心点坐标
        position.x = leftUpX + len / 2 + (j % num) * len;
        position.y = leftUpY - len / 2 - Math.floor(j / num) * len;
        position.z = leftUpZ - len / 2 - i * len;
        cubes.push(
          <mesh
            position={[position.x, position.y, position.z]}
            material={materials}
            key={`${i}-${j}`}
          >
            <boxGeometry
              args={[BasicParams.len, BasicParams.len, BasicParams.len]}
            />
          </mesh>
        );
      }
    }
    return cubes;
};
```


>  
> 
> 这里需要特别注意的一点是，其实对于一个魔方，不管它是二阶、三阶还是N阶、每个面的颜色以及大小怎样，其实生成和旋转的逻辑都是一样的。我门就可以将魔方的属性参数提取成一个常量对象，这样就可以通过配置实现阶数、大小以及魔方颜色的快速变换。
> 
> ```js
> //基础模型参数
> const BasicParams = {
>   x: 0,
>   y: 0,
>   z: 0, 
>   num: 2, // 阶数
>   len: 1, // 小方块大小
>   //右、左、上、下、前、后
>   colors: ["#ff6b02", "#dd422f", "#ffffff", "#fdcd02", "#3d81f7", "#019d53"], // 每面的颜色
> };
> ```
> 
#### 添加材质

上面的代码已经将魔方的骨架搭建好了，接下来我们再根据在每个面的颜色生成对应的[纹理 (Texture)](https://link.juejin.cn?target=https%3A%2F%2Fthreejs.org%2Fdocs%2F%23api%2Fzh%2Ftextures%2FTexture) ，并将这些纹理分别添加到[材质 (Material)](https://link.juejin.cn?target=https%3A%2F%2Fthreejs.org%2Fdocs%2F%23api%2Fzh%2Fmaterials%2FMaterial) 中，这里我们先选用的使用最简单的材质`MeshBasicMaterial`， 这种材质不是基于物理的，也就是说不需要灯光也能正常显示，缺点就是会缺少一些真实感，但对于初步实现一个魔方是最简单方便的选择。

```jsx
const faces = (rgbaColor: string) => {
	const canvas = document.createElement("canvas");
	canvas.width = 256;
	canvas.height = 256;
	const context = canvas.getContext("2d");
	if (!context) return;
	//画一个宽高都是256的黑色正方形
	context.fillStyle = "rgba(0,0,0,1)";
	context.fillRect(0, 0, 256, 256);
	//在内部用某颜色的16px宽的线再画一个宽高为224的圆角正方形并用改颜色填充
	context.rect(16, 16, 224, 224);
	context.lineJoin = "round";
	context.lineWidth = 16;
	context.fillStyle = rgbaColor;
	context.strokeStyle = rgbaColor;
	context.stroke();
	context.fill();
	return canvas;
};
```

```jsx
const genMaterials = () => {
	const myFaces = [];
	for (let k = 0; k < 6; k++) {
	  myFaces[k] = faces(BasicParams.colors[k]);
	}
	const materials = [];
	for (let k = 0; k < 6; k++) {
	  const texture = new THREE.Texture(myFaces[k]);
	  texture.needsUpdate = true;
	  materials.push(new THREE.MeshBasicMaterial({ map: texture }));
	}
	return materials;
};
```

### 魔方转动

#### 事件监听

让魔方转动的整体实现思路是：首先，对鼠标（网页）和触摸（移动端）事件进行监听，在startMouse中获取到用户的点击的起始点，然后再在 moveMouse 中获取到当前移动的点，接着就是计算两个点之间的向量，从而得到用户的旋转方向。最后，根据旋转方向获取到需要旋转的小方块，执行滚动动画。

```js
const setupEvents = () => {
    window.addEventListener("mousedown", startMouse);
    window.addEventListener("mousemove", moveMouse);
    window.addEventListener("mouseup", stopMouse);
    window.addEventListener("touchstart", startMouse);
    window.addEventListener("touchmove", moveMouse);
    window.addEventListener("touchend", stopMouse);
  };

  /**
   * 魔方控制方法
   */
  function startMouse(event: any) {
    // 找到
    const value = getIntersectAndNormalize(event);
    normalize = value.normalize;
    // 魔方没有处于转动过程中且存在碰撞物体
    if (!isRotating && value.intersect) {
      // controller.enabled = false; // 当刚开始的接触点在魔方上时操作为转动魔方，屏蔽控制器转动
      startCube = value.intersect.object;
      startPoint = value.intersect.point; // 开始转动，设置起始点
    } else {
      // controller.enabled = true; // 当刚开始的接触点没有在魔方上或者在魔方上，但是魔方正在转动时操作转动控制器
    }
  }

  function moveMouse(event: any) {
    const value = getIntersectAndNormalize(event);
    if (!isRotating && value.intersect && startPoint) {
      const movePoint = value.intersect.point;
      if (!movePoint.equals(startPoint)) {
        isRotating = true;
        let vector = movePoint.sub(startPoint);
        let direction = getDirection(vector);
        console.log("direction", direction);
        let cubes = getPlaneCubes(startCube, direction);
        requestAnimationFrame((timestamp) => {
          rotateAnimation(cubes, direction, timestamp, 0, 0);
        });
      }
    }
  }

  function stopMouse(event: any) {
    startCube = null;
    startPoint = null;
    isRotating = false;
    // controller.enabled = true;
  }

  useEffect(() => {
    setupEvents();
  }, []); // 不会再次运行（开发环境下除外）
```

为了简化事件处理，在魔方的外部包围一个透明的并且和魔方一样大小的立方体，作为事件的代理对象。先给这个透明立方体起一个名称叫做`coverCube`，后面就可以直接通过名字获取到它。

```js
const getOpacityRubik = () => {
	// 透明正方体
	const size = BasicParams.len * BasicParams.num;
	const material = new THREE.MeshBasicMaterial({
	  opacity: 0,
	  transparent: true,
	  // color: "red",
	});
	return (
	  <mesh material={material} name="coverCube" key="coverCube">
		<boxGeometry args={[size, size, size]} />
	  </mesh>
	);
};
```

#### 获取触摸点

下面详细讲讲具体实现，首先是获取用户触摸点的方法。

```js

  /**
   * 获取操作焦点以及该焦点所在平面的法向量
   * */
  function getIntersectAndNormalize(event: any) {
    let mouse = new THREE.Vector2();
    if (event.touches) {
      // 触摸事件
      const touch = event.touches[0];
      mouse.x = (touch.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(touch.clientY / window.innerHeight) * 2 + 1;
    } else {
      // 鼠标事件
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    }

    const raycaster = new THREE.Raycaster();
    raycaster.setFromCamera(mouse, camera);
    // Raycaster方式定位选取元素，可能会选取多个，以第一个为准
    const intersects = raycaster.intersectObjects(scene.children);
    let intersect, normalize;
    if (intersects.length) {
      try {
        if (intersects[0].object?.name === "coverCube") {
          intersect = intersects[1];
          normalize = intersects[0].face?.normal;
        } else {
          intersect = intersects[0];
          normalize = intersects[1].face?.normal;
        }
      } catch (err) {
        //nothing
      }
    }
    console.log("normalize", normalize);
    return { intersect: intersect, normalize: normalize };
  }
```

这个方法根据当前用户的点击事件获取坐标点，分别对触摸和鼠标操作进行了兼容性处理，获取到二维坐标点。

然后再借助Three.js自带的Raycaster 工具，通过光线投射的方式计算出鼠标在三维场景中经过的点，经过的点可能不止一个，我们只选取第一个叫做`coverCube` 的元素，返回三维空间中点所在平面的法向量。

#### 计算旋转方向

通过两个点计算出一个三维向量，找到与坐标轴最小的夹角，确定大致方向之后再进一步细化方向。

![ThreeJS实现网页版魔方 20240809092352044](https://image.songxingguo.com/obsidian/20240809/ThreeJS%E5%AE%9E%E7%8E%B0%E7%BD%91%E9%A1%B5%E7%89%88%E9%AD%94%E6%96%B9-20240809092352044.webp)

```jsx
// 魔方转动的六个方向
  const xLine = new THREE.Vector3(1, 0, 0); // X轴正方向
  const xLineAd = new THREE.Vector3(-1, 0, 0); // X轴负方向
  const yLine = new THREE.Vector3(0, 1, 0); // Y轴正方向
  const yLineAd = new THREE.Vector3(0, -1, 0); // Y轴负方向
  const zLine = new THREE.Vector3(0, 0, 1); // Z轴正方向
  const zLineAd = new THREE.Vector3(0, 0, -1); // Z轴负方向

  /**
   * 获得旋转方向
   * vector3: 鼠标滑动的方向
   */
  function getDirection(vector3: any) {
    let direction;
    // 判断差向量和 x、y、z 轴的夹角
    const xAngle = vector3.angleTo(xLine);
    const xAngleAd = vector3.angleTo(xLineAd);
    const yAngle = vector3.angleTo(yLine);
    const yAngleAd = vector3.angleTo(yLineAd);
    const zAngle = vector3.angleTo(zLine);
    const zAngleAd = vector3.angleTo(zLineAd);
    const minAngle = Math.min(
      ...[xAngle, xAngleAd, yAngle, yAngleAd, zAngle, zAngleAd]
    ); // 最小夹角
    switch (minAngle) {
      case xAngle:
        direction = 10; // 向x轴正方向旋转90度（还要区分是绕z轴还是绕y轴）
        if (normalize.equals(yLine)) {
          direction = direction + 5; // 绕z轴顺时针
        } else if (normalize.equals(yLineAd)) {
          direction = direction + 6; // 绕z轴逆时针
        } else if (normalize.equals(zLine)) {
          direction = direction + 4; // 绕y轴逆时针
        } else if (normalize.equals(zLineAd)) {
          direction = direction + 3; // 绕y轴顺时针
        }
        break;
      case xAngleAd:
        direction = 20; // 向x轴反方向旋转90度
        if (normalize.equals(yLine)) {
          direction = direction + 6; // 绕z轴逆时针
        } else if (normalize.equals(yLineAd)) {
          direction = direction + 5; // 绕z轴顺时针
        } else if (normalize.equals(zLine)) {
          direction = direction + 3; // 绕y轴顺时针
        } else if (normalize.equals(zLineAd)) {
          direction = direction + 4; // 绕y轴逆时针
        }
        break;
      case yAngle:
        direction = 30; // 向y轴正方向旋转90度
        if (normalize.equals(zLine)) {
          direction = direction + 1; // 绕x轴顺时针
        } else if (normalize.equals(zLineAd)) {
          direction = direction + 2; // 绕x轴逆时针
        } else if (normalize.equals(xLine)) {
          direction = direction + 6; // 绕z轴逆时针
        } else {
          direction = direction + 5; // 绕z轴顺时针
        }
        break;
      case yAngleAd:
        direction = 40; // 向y轴反方向旋转90度
        if (normalize.equals(zLine)) {
          direction = direction + 2; // 绕x轴逆时针
        } else if (normalize.equals(zLineAd)) {
          direction = direction + 1; // 绕x轴顺时针
        } else if (normalize.equals(xLine)) {
          direction = direction + 5; // 绕z轴顺时针
        } else {
          direction = direction + 6; // 绕z轴逆时针
        }
        break;
      case zAngle:
        direction = 50; // 向z轴正方向旋转90度
        if (normalize.equals(yLine)) {
          direction = direction + 2; // 绕x轴逆时针
        } else if (normalize.equals(yLineAd)) {
          direction = direction + 1; // 绕x轴顺时针
        } else if (normalize.equals(xLine)) {
          direction = direction + 3; // 绕y轴顺时针
        } else if (normalize.equals(xLineAd)) {
          direction = direction + 4; // 绕y轴逆时针
        }
        break;
      case zAngleAd:
        direction = 60; // 向z轴反方向旋转90度
        if (normalize.equals(yLine)) {
          direction = direction + 1; // 绕x轴顺时针
        } else if (normalize.equals(yLineAd)) {
          direction = direction + 2; // 绕x轴逆时针
        } else if (normalize.equals(xLine)) {
          direction = direction + 4; // 绕y轴逆时针
        } else if (normalize.equals(xLineAd)) {
          direction = direction + 3; // 绕y轴顺时针
        }
        break;
      default:
        break;
    }
    return direction;
  }
```

#### 获取方块

紧接着，根据旋转方向对10取余，得到旋转轴，并将对应平面的小方块加入到需要旋转的数组中。

```js
/**
   * 根据立方体和旋转方向，找到同一平面上的所有立方体
   */
function getPlaneCubes(cube: any, direction: any) {
    const cubes = scene.children;
    let results = [];
    let orientation = direction % 10;
    switch (orientation) {
      case 1:
      case 2:
        // 绕x轴
        for (let i = 0; i < cubes.length; i++) {
          let curr = cubes[i];
          // console.log("绕x轴", curr.position, cube.position);
          if (Math.abs(curr.position.x - cube.position.x) < 0.2) {
            results.push(curr);
          }
        }
        break;
      case 3:
      case 4:
        // 绕y轴
        for (let i = 0; i < cubes.length; i++) {
          let curr = cubes[i];
          // console.log("绕y轴", curr.position, cube.position);
          if (Math.abs(curr.position.y - cube.position.y) < 0.2) {
            results.push(curr);
          }
        }
        break;
      case 5:
      case 6:
        // 绕z轴
        for (let i = 0; i < cubes.length; i++) {
          let curr = cubes[i];
          // console.log("绕z轴", curr.position, cube.position);
          if (Math.abs(curr.position.z - cube.position.z) < 0.2) {
            results.push(curr);
          }
        }
        break;
    }
    return results;
}
```

#### 旋转动画

最后，同样是获取取余获得旋转轴，再根据旋转轴进一步对2取余的得到正（反）旋转方向，进而得到旋转角度。最后，根据旋转轴和旋转角度进行旋转。

```jsx
  function rotateAnimation(
    cubes: any,
    direction: any,
    currentstamp: any,
    startstamp: any,
    laststamp: any
  ) {
    if (startstamp === 0) {
      startstamp = currentstamp;
      laststamp = currentstamp;
    }
    if (currentstamp - startstamp >= rotateDuration) {
      currentstamp = startstamp + rotateDuration;
      isRotating = false;
      startPoint = null;
    }
    let orientation = direction % 10;
    let radians = orientation % 2 == 1 ? -90 : 90; // 正/反转
    const rotationRatio = (currentstamp - laststamp) / rotateDuration; //旋转比率
    const tRotationAngle = (radians * Math.PI) / 180; //总旋转角度
    const rotationAngle = tRotationAngle * rotationRatio; // 每次旋转角度
    switch (orientation) {
      case 1:
      case 2:
        for (let i = 0; i < cubes.length; i++) {
          rotateAroundWorldX(cubes[i], rotationAngle);
        }
        break;
      case 3:
      case 4:
        for (let i = 0; i < cubes.length; i++) {
          rotateAroundWorldY(cubes[i], rotationAngle);
        }
        break;
      case 5:
      case 6:
        for (let i = 0; i < cubes.length; i++) {
          rotateAroundWorldZ(cubes[i], rotationAngle);
        }
        break;
    }
    if (currentstamp - startstamp < rotateDuration) {
      requestAnimationFrame((timestamp) => {
        rotateAnimation(cubes, direction, timestamp, startstamp, currentstamp);
      });
    }
  }

  function rotateAroundWorldX(cube: any, rad: any) {
    const y0 = cube.position.y;
    const z0 = cube.position.z;
    const q = new THREE.Quaternion();
    q.setFromAxisAngle(new THREE.Vector3(1, 0, 0), rad);
    cube.quaternion.premultiply(q);
    cube.position.y = Math.cos(rad) * y0 - Math.sin(rad) * z0;
    cube.position.z = Math.cos(rad) * z0 + Math.sin(rad) * y0;
  }

  function rotateAroundWorldY(cube: any, rad: any) {
    const x0 = cube.position.x;
    const z0 = cube.position.z;
    const q = new THREE.Quaternion();
    q.setFromAxisAngle(new THREE.Vector3(0, 1, 0), rad);
    cube.quaternion.premultiply(q);
    cube.position.x = Math.cos(rad) * x0 + Math.sin(rad) * z0;
    cube.position.z = Math.cos(rad) * z0 - Math.sin(rad) * x0;
  }

  function rotateAroundWorldZ(cube: any, rad: any) {
    const x0 = cube.position.x;
    const y0 = cube.position.y;
    const q = new THREE.Quaternion();
    q.setFromAxisAngle(new THREE.Vector3(0, 0, 1), rad);
    cube.quaternion.premultiply(q);
    cube.position.x = Math.cos(rad) * x0 - Math.sin(rad) * y0;
    cube.position.y = Math.cos(rad) * y0 + Math.sin(rad) * x0;
  }
```

