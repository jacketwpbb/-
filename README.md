### 微信小程序总结

#### 微信小程序ajax请求封装
微信小程序自身提供了`wx.request`接口来进行网络请求，基于这个我们来封装一个基于Promise的函数，并处理请求状态，在请求失败时提示是否刷新页面

使用方法
```javascript
//customPage.js
import http from "../../utils/http.js";
Page({

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function({ house_id }) {
    http
      .get("v4_0/house/detail", {
        //参数
		id:111
      })
      .then(res => {
        console.log(res);
      });
  }
})
```





首先我们规定后台返回特定格式的数据，请求信息由后台返回
```javascript
//这里我们规定后端的接口返回特定格式的数据
//示例
{
	//规定code为100时正常请求成功，其他情况请求异常
	code:100,
	//后台请求返回的信息，code不是100时，调用wx.showToast展示的错误信息
	msg:'请求成功',
	//返回的数据
	data:{
		//这里是数据
	}
}
```

封装get请求，根据定义的接口获取，当请求错误时提示是否刷新页面

```javascript
//http.js
import { reloadCurrentPgae } from "util.js";
const app = getApp();
const default_token = "2IYwitW3vhhzhHOvLSxiyNTWceCj0swKjsKyh9JG";
//请求的基础URL
const BASE_URL = "https://api.xxx.com/";
/**
 * GET请求
 * @param  {[String]} path [请求的路径]
 * @param  {[Object]} data [description]
 */
const getRequest = (path, data) =>
	new Promise((resolve, reject) => {
		//根据情况为每次请求加入自定义数据
		//这里我们给每条请求加一个token
		//并带上wechat标记
		const access_token = app.globalData.token || default_token;

		data = { ...data, access_token, wechat: "helper" };

		wx.request({
			url: BASE_URL + path,
			data,
			method: "GET",
			success(res) {
				const { code, msg = "获取错误", data } = res.data;
				// 如果没有正常请求成功
				// 根据后台规定的接口，code不是100就是非正常请求，展示输出返回的错误信息
				// 如果接口没有返回错误信息，则输出默认值： 获取错误
				if (code !== 100) {
					wx.showToast({
						title: msg,
						icon: "none",
						duration: 2000
					});
					return;
				} else {
					//请求成功
					resolve(data);
				}
			},
			//连接失败
			fail(error) {
				console.error(error);
				reject(error);
				wx.showModal({
					title: "提示",
					content: "网络错误,是否刷新页面？",
					success: function(res) {
						//点击确定
						if (res.confirm) {
							//刷新页面
							reloadCurrentPgae();
						} else if (res.cancel) {
						}
					}
				});
			}
		});
	});

export default {
	get: getRequest
};


```

实现刷新页面：获取当前页面页面url并调用wx.redirectTo
```javascript
//util.js
/*获取当前页url*/
function getCurrentPageUrl() {
  var pages = getCurrentPages(); //获取加载的页面
  var currentPage = pages[pages.length - 1]; //获取当前页面的对象
  var url = currentPage.route; //当前页面url
  return url;
}

/*获取当前页带参数的url*/
function getCurrentPageUrlWithArgs() {
  var pages = getCurrentPages(); //获取加载的页面
  var currentPage = pages[pages.length - 1]; //获取当前页面的对象
  var url = currentPage.route; //当前页面url
  var options = currentPage.options; //如果要获取url中所带的参数可以查看options

  //拼接url的参数
  var urlWithArgs = url + "?";
  for (var key in options) {
    var value = options[key];
    urlWithArgs += key + "=" + value + "&";
  }
  urlWithArgs = urlWithArgs.substring(0, urlWithArgs.length - 1);

  return urlWithArgs;
}

/*刷新页面*/
function reloadCurrentPgae() {
  const url = getCurrentPageUrlWithArgs();
  wx.redirectTo({
    url: "/" + url
  });
}

module.exports = {
  getCurrentPageUrl,
  getCurrentPageUrlWithArgs,
  reloadCurrentPgae
};

```
