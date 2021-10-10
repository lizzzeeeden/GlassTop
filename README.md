# GlassTop
偶然间萌生了想把游戏叙事课上写的交互ppt做成网页的想法，于是开启了html之旅  
非正常学习道路的速成，代码上写得很不规范，但竟然也拼拼凑凑着完成了网页移植  

github pages嵌入到项目的settings中之后托管网站方便了许多，简单记录一下网页交互的实现  

首先这篇交互故事的结构是分支型的，为了控制分支体量，每次的分支选项至多三个。因为文本内容较多，在制作ppt时一页的容纳量需要控制，因此有些衔接页至少会有一个选项。  
所有的选项用三个按钮实现，每次选择之后重新给这三个按钮赋值即可。  

每个选项的结构如下：
```js
index: 0,
content: "", //选项上的文字
toPage: 0, //点击后跳转至的页面
```

每次做出选择之后显示一个ppt页的故事文本
每页的结构如下：
```js
index: 0,
ownBtns: [0, 0, 0], //每页含有的按钮序号，0号为空
content: [""],//每页的故事文本
endColor: "rgb(255,255,255)",//若为结局页则标记结局颜色（依照流程图共有三种结局颜色）
fontSize: 0,//字体大小，为一些特殊页添加的特殊效果，默认则缺省
```
每一页、每一个按钮的参数都需要手动设置，体量大的话非常麻烦。  
为了使文本分段，在页结构的content中每一段都需要新加一个字符串成员，因为文本量巨大，还使用unity做了个辅助脚本来转换文本……  
需要找到一个优化方法。  

页面初始化时为按钮附上监听事件  
```js
for (let n = 0; n < 3; n++) {
    btns[n].onclick = function() {
		  //特殊按键（一些特殊剧情点）
		  switch (currBtns[n]) {    //注意这里的currBtns是动态的，会随着onclick更新
			  //学习种蘑菇（该选项可影响后续的剧情，ppt中无法实现这个需求）
			  case 9:
				  bLearnMushroom = true;
				break;
				//再演
		  	case 131:
				  location.reload();
			  break;
		 }

		//赋上新页参数
		currPage = btnsContent[page[currPage].ownBtns[n]].toPage;
		//特殊页
		switch (currPage) {
			//可选种蘑菇页
			case 27:
				if (bLearnMushroom) {
					page[currPage].ownBtns[1] = 44;
				}
			break;
		}
		//更新按钮内容
		currBtns = page[currPage].ownBtns;
		for (let i = 0; i < 3; i++) {
			if (page[currPage].ownBtns[i] == 0) {
				btns[i].style.display = "none";
			} else {
				btns[i].style.display = "block";
				btns[i].innerHTML = btnsContent[page[currPage].ownBtns[i]].content;
			}
		}

		//文本显示
    //在html中有一个id为story的块，所有的故事文本在该块中根据选项动态生成
    
		//水平分隔线
		let hr = document.createElement("hr");
		storyContent.appendChild(hr);
		//特殊页显示
		switch (currPage) {
			//剧终标题
			case 84:
				let h2 = document.createElement("h2"); //新建段落
				h2.innerText = "剧终";
				storyContent.appendChild(h2);
			break;
		}
		//前空行
		let p1 = document.createElement("p");
		p1.innerText = "\n";
		storyContent.appendChild(p1);
		//显示当前页内容
		for (let i = 0; i < page[currPage].content.length; i++) {
			let p = document.createElement("p"); //新建段落
			p.innerText += page[currPage].content[i];
			storyContent.appendChild(p);

			//特殊字体大小
			if (page[currPage].fontSize != undefined) {
				p.style.fontSize = page[currPage].fontSize + 'rem';
			}
		}
		//后空行
		let p2 = document.createElement("p");
		p2.innerText = "\n";
		storyContent.appendChild(p2);
		//end背景色
		if (page[currPage].endColor != undefined) {
			document.body.style.backgroundColor = page[currPage].endColor;
		}
    
    //点击后窗口滚动到新文本处
		window.scrollTo({
			top: hr.offsetTop,
			behavior: 'smooth',
		})
	};
}
```
