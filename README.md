# react-img-lazyload
仅仅对react图片进行懒加载的组件。

## 特点

- 性能突出，所有组件仅仅监听两个事件
- 使用函数节流，`scroll`、`resize`两个事件处理并不会频繁

## 使用

```
import React ,{Component} from 'react';
import ImgLazy from 'ImgLazy';

class Demo extends Component{

	render(){
		return(
			<div>
				<ImgLazy src="http://img.png"
						placeholder="http://默认图片.png"
						offset={100} />
			</div>
		)
	}
}

```
当 `ImgLazy`在可视范围时，正在需要显示的图片才会进行加载。

## Props
- **src**   
字符串地址。真正需要显示图片的地址  


- **placeholader**  
字符串地址。在真正图片未显示时，会显示该地址的图片

- **offset**
数值。设置图片离可视范围多少时提前加载

## 源码
一、   
通过内部状态的 `isLoaded`来控制真正的图片是否加载。当图片在可视范围时设置 `isLoaded = true`，这样，无论外部传递的 `props.src` 是否有改变，都会render。   

**说明**  
初次编写时，是使用内部状态来保持需要显示的图片地址。在未到可视范围时，state是 `props.placeholder`，而达到可视范围时则是 `props.src`。正常来说是没问题，但是遇到第一次传入的 `src`并不是真正的图片地址时（譬如该地址需要从服务器请求才获得，但是这是图片已经在可视范围内了），则会有问题。

二、   
我们都知道，在组件挂载时需要监听 `scroll`和 `resize`事件；而图片在可视范围后需要解除事件绑定。  
但是很容易忘记，在组件被移除时，也需要解除事件绑定。   

三、   
 `scroll` 和 `resize` 这两个事件，触发相当的频繁，必须使用函数节流来维持性能

```
/**
 * React版的图片懒加载
 */

import React ,{Component} from 'react';
import ReactDOM from 'react-dom';

export default class ImgLazy extends Component {
    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {
            isLoaded: false,
        };
        this._handleScroll = this._handleScroll.bind(this);
        //使用函数节流，性能优先
        this.handleScroll = this.throttle(this._handleScroll, 100);
    }

    /**
     * 添加监听事件
     */
    componentDidMount() {
        window.addEventListener('scroll', this.handleScroll);
        window.addEventListener('resize', this.handleScroll);
        this._handleScroll();
    }

    /**
     * 移除事件
     */
    componentWillUnMount() {
        window.removeEventListener('scroll', this.handleScroll);
        window.removeEventListener('resize', this.handleScroll);
    }

    /**
     * 获取窗口的高度
     * @returns {XML}
     */
    getClientHeight() {
        var clientHeight = 0;
        if (document.body.clientHeight && document.documentElement.clientHeight) {
            clientHeight = Math.min(document.body.clientHeight, document.documentElement.clientHeight);
        } else {
            clientHeight = Math.max(document.body.clientHeight, document.documentElement.clientHeight);
        }
        return clientHeight;
    }

    /**
     * 获取滚动条滚动的高度
     */
    getScrollTop() {
        let scrollTop = 0;
        if (document.documentElement && document.documentElement.scrollTop) {
            scrollTop = document.documentElement.scrollTop;
        } else if (document.body) {
            scrollTop = document.body.scrollTop;
        } else {
            scrollTop = window.scrollY || window.pageYOffset;
        }

        return scrollTop;
    }

    /**
     * 获取当前图片距离顶部的xy坐标
     */
    getNodeTop() {
        const viewTop = this.getScrollTop();

        const img = ReactDOM.findDOMNode(this); //当前节点
        const nodeTop = img.getBoundingClientRect().top + viewTop;
        const nodeBottom = nodeTop + img.offsetHeight;
        return {
            nodeTop: nodeTop,
            nodeBottom: nodeBottom,
        };
    }


    /**
     * 函数节流
     * @returns {XML}
     */
    throttle(fn, delay) {
        let timer = null;

        return function () {
            let context = this;
            let args = arguments;
            clearTimeout(timer);
            timer = setTimeout(function () {
                fn.apply(context, args);
            }, delay);
        }
    }

    /**
     * 处理滚动事件
     * @returns {XML}
     */
    _handleScroll() {
        const {offset} = this.props; //偏移量

        const {nodeTop ,nodeBottom} = this.getNodeTop();

        const viewTop = this.getScrollTop();
        const viewBottom = viewTop + this.getClientHeight();

        //当图片出现在视野范围内,设置真正的图片，同时移除监听
        if (nodeBottom + offset >= viewTop && nodeTop - offset <= viewBottom) {
            this.setState({
                isLoaded: true,
            });
            window.removeEventListener('scroll', this.handleScroll);
            window.removeEventListener('resize', this.handleScroll);
        }
    }

    render() {
        const {
            className,
            style,
            src,
            placeholder} = this.props;
        const {isLoaded} = this.state;
        let true_src = isLoaded ? src : placeholder;
        return (
            <img className={className} style={style}
                 src={true_src}/>
        )
    }
}

ImgLazy.defaultProps = {
    placeholder: 'https://sf-static.b0.upaiyun.com/v-583bf474/global/img/user-64.png',//默认图片
    offset: 0,//默认距离
}

```

## 相关总结
- [按需加载之图片懒加载](https://github.com/guanMac/blog/blob/master/web%E5%89%8D%E7%AB%AF/%E6%8C%89%E9%9C%80%E5%8A%A0%E8%BD%BD%E4%B9%8B%E5%9B%BE%E7%89%87%E6%87%92%E5%8A%A0%E8%BD%BD.md)
- [按需加载之函数节流](https://github.com/guanMac/blog/blob/master/web%E5%89%8D%E7%AB%AF/%E6%8C%89%E9%9C%80%E5%8A%A0%E8%BD%BD%E4%B9%8B%E5%87%BD%E6%95%B0%E8%8A%82%E6%B5%81.md)
- [按需加载之滚动加载](https://github.com/guanMac/blog/blob/master/web%E5%89%8D%E7%AB%AF/%E6%8C%89%E9%9C%80%E5%8A%A0%E8%BD%BD%E4%B9%8B%E6%BB%9A%E5%8A%A8%E5%8A%A0%E8%BD%BD.md)


## 其他相关组件
[react-lazyload](https://github.com/jasonslyvia/react-lazyload)