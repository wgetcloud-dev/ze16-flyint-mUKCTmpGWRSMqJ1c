
## 前言


不知不觉已经九月了，又到了一年的开学季，我每年都想做的项目墙甚至连个影子都没有…😂


最近生活中的琐事太多了，导致完全没有想写文章的动力，不过再怎么拖还是得记录，随便写写吧\~


这次是7月份的一个小项目，谈不上什么技术含量，算是友情开发了。后端 DjangoStarter v3，前端使用 Taro 做微信小程序。


事实上后端基本没啥好说的，基本有什么坑能记录的都在小程序这块，不得不说微信真是史中史，不仅代码写得跟答辩一样，文档更是一坨😂


之前看网上有个说法，正因为微信做得跟答辩一样，才给了小程序开发者一道护城河，现在想起来还蛮有道理的，哈哈哈😃


## 后端部分


先来说说后端部分吧


自从有了 DjangoStarter ，这类小项目的后端开发已经直接秒了


这个项目里用到几个新的，我之前有写文章介绍了，直接放链接吧\~


* [Django里集成腾讯COS对象存储](https://github.com)
* [使用 django\-treebeard 编辑类别](https://github.com)


### 使用 django\-filer 管理文件


忘了这个还没写文章，那么简单介绍一下吧，这个组件可以实现文件（包括图片）的集中管理，还提供了 admin 界面，不过UI风格就比较原始了，而且中文locale还有bug，得手动修改才能正常切换到中文。不过对于这个项目来说是够用的


项目地址: [https://github.com/django\-cms/django\-filer](https://github.com)


安装: `pdm add django-filer`


把 `filer` 添加到 `INSTALLED_APPS` 里


添加 `src/config/settings/components/filer.py` 配置



```
THUMBNAIL_PROCESSORS = (
    'easy_thumbnails.processors.colorspace',
    'easy_thumbnails.processors.autocrop',
    'easy_thumbnails.processors.scale_and_crop',
    # Subject location aware cropping
    # 'filer.thumbnail_processors.scale_and_crop_with_subject_location',
    'easy_thumbnails.processors.filters',
)

FILER_ENABLE_LOGGING = True

```

最后修改一下需要用到附件的字段



```
from filer.fields.image import FilerImageField

class Brand(ModelExt):
    name = models.CharField('名称', max_length=100)
    # logo = models.ImageField(upload_to='brand_logos/')
    logo = FilerImageField(
        verbose_name='Logo', null=True, blank=True,
        related_name='+', on_delete=models.SET_NULL,
    )

```

大概就这样，具体用法看官方文档吧\~


### 使用 ninja 来编写API


DjangoStarter v3 开始从drf 切换到 ninja


这个 ninja 和 FastAPI 非常像，写起来比 drf 舒服多了，虽然不能像 drf 一样自动生成 crud 接口，不过写起来也多不了多少代码，而且更加灵活。


（PS：DjangoStarter v3 依然支持自动生成 crud 接口，同时还可以使用第三方库来实现自动生成基于 ninja 的 crud 接口，不过我还没用过，可以参考下面的扩展部分）


比如上面说的 django\-filer 库，图片字段其实是指向 `filer.fields.models.File` 模型的外键，所以接口如果没做处理，生成出来的数据就只是一个外键ID而已，所以要修改一下 schema



```
class CaseOut(ModelSchema):
  cover_image: str

  @staticmethod
  def resolve_cover_image(obj: Case):
    if not obj.cover_image:
      return f'https://starblog/Api/PicLib/Random/{obj.id}/250/150'
    return obj.cover_image.url

  class Meta:
    model = Case
    fields = [
      'id', 'is_deleted', 'created_time', 'updated_time', 'name', 'description',
      'category', 'car_model', 'part', 'build_time',
    ]

```

以上代码会把 `cover_image` 字段渲染为一个图片地址，如果 `cover_image` 不存在的话，则返回一个随机图片地址。


schema 实现了逻辑和数据渲染分开，代码结构更清晰。


还可以实现更复杂的逻辑，只需要实现 `resolve_字段名` 方法就行。


### ninja 生态扩展


我也是最近才发现的，原来 django\-ninja 还有个中文网，里面写了几个 ninja 生态的扩展，感觉都挺不错的，以后有空可以试试。


网址: [https://django\-ninja.cn/](https://github.com)


* Ninja JWT \- 一个用于 Django Ninja REST 框架的 JSON Web 令牌认证插件。
* Django Ninja Extra \- Django Ninja Extra 提供了一种 基于类 的方法以及额外的功能，这将使用 Django Ninja 加速您的 RESTful API 开发。
* Django Ninja CRUD \- Django Ninja CRUD 是一个强大的、声明式的、但又有点固执己见的框架，它简化了使用 Django Ninja 开发 CRUD（创建、读取、更新、删除）端点的过程，并且还提供了一种声明式的基于场景的方法，用于使用 Django REST Testing（这个包的小弟）测试这些端点。


## 小程序部分


这次依然使用 Taro 来开发移动端，上次是做 H5（公众号），这次试试 Taro 做出来的小程序咋样，实际效果还行。


状态管理依然选择了 mobx ，用习惯了，前端轮子太多，懒得去试用其他的了\~


相比起后端部分，小程序能写的东西会多一点点（但也不多，都很简单）


### 在Taro里使用tailwindcss


最近我开始使用 tailwindcss ，一下就喜欢上这种高效的样式工具（虽然会有很长的一串class）不过瑕不掩瑜，使用 tailwindcss 可以很方便在网络上 copy 各种样式，还能让 LLM 帮我写各种样式，生产力拉满了\~


Taro 官方提供了 tailwindcss 的支持，这点非常好，跟着官方文档来就行


详见官方文档: [https://docs.taro.zone/docs/tailwindcss](https://github.com)


### 分享小程序


如果不主动调用，那么只有注册了 `onShareAppMessage` 事件，才能在点右上角三个点的时候，显示转发按钮；同样的，注册了 `onShareTimeline` 事件才能分享到朋友圈。


相应的，Taro 里提供了这俩事件的 hook ，直接看代码



```
import Taro, {useShareAppMessage, useShareTimeline} from '@tarojs/taro'

const CasePage = () => {   
  const getCase = async () => {
    if (!id) return
    const data = await CaseService.get(Number(id))
    console.log('get case', data)
    setCaseData(data.data)
    return {
      title: `案例：${data.data.name}`,
      path: RouterMap.car.case(data.data.id),
      imageUrl: data.data.cover_image
    }
  }

  useShareAppMessage(res => {
    console.log('执行分享操作', res)
    return {
      title: '案例分享',
      path: RouterMap.index,
      promise: new Promise(resolve => getCase().then((data) => {
        // @ts-ignore
        resolve(data)
      })),
    }
  })

  useShareTimeline(() => {
    console.log('执行分享朋友圈操作')

    return {
      title: `案例：${caseData.name}`,
      query: `id=${caseData.id}`,
      imageUrl: caseData.cover_image
    }
  })
}

```

跟小程序文档里说的一样的，需要返回的参数啥的也一样，所以直接看文档吧\~


#### 主动分享


主动分享的话，只要把按钮设置成 `share` 类型就行，Taro 同样做了包装。



```
<Button type='info' icon={<Share/>} openType='share'>分享Button>

```

#### 参考资料


* [https://juejin.cn/post/7261774602481369147](https://github.com):[westworld加速](https://tianchuang88.com)
* [https://developers.weixin.qq.com/miniprogram/dev/framework/open\-ability/share\-timeline.html](https://github.com)
* [https://developers.weixin.qq.com/miniprogram/dev/reference/api/Page.html\#onShareAppMessage\-Object\-object](https://github.com)


### 用户登录


本项目中我是做一个单独的页面来处理登录，也可以用 Modal 的形式，不过一开始我还对获取用户信息抱有幻想，因为文档介绍获取用户头像、昵称等信息需要主动触发才行，所以放在一个单独的页面，提供一个登录按钮，让用户去点击。


不过后面发现新版本的小程序已经不能用这种方式获取信息了……


登录这块 Taro 也封装好了，其实就是把 `wx.login` 的 wx 改成 Taro 而已



```
const LoginPage = observer(() => {
  const handleLogin = async (data: LoginToken) => {
    UserStore.login({
      token: data.token,
      exp: String(data.exp),
    })

    const userInfo = await AccountService.getCurrentUser()
    console.log('获取用户信息', userInfo)
    UserStore.userInfo = userInfo.data
    Taro.navigateBack()
  }

  const autoLogin = async () => {
    Taro.showToast({title: '正在登录', icon: 'loading'})
    Taro.login({
      success: async (res) => {
        if (res.code) {
          console.log('小程序登录，获取code', res.code)
          const resp = await AccountService.loginWeApp(res.code)
          console.log('小程序登录，请求后端', resp)
          await handleLogin(resp.data)
        } else {
          console.log('登录失败！', res.errMsg)
        }
      }
    })
  }
}

```

拿到小程序 OAuth 之后的 code，调用后端接口登录，搞定。


而且这部分 DjangoStarter v3 也集成了🕶


### 点击图片打开大图


在图片组件的 `onTap` 事件里调用 `Taro.previewImage` 就行，记得把大图的地址传入（如果小图是单独的地址的话）



```
<Image
  key={e.id} src={e.url}
  onTap={() => {
    Taro.previewImage({
      current: e.url,
      urls: caseData.images.map(i => i.url)
    })
  }}
  className="w-full rounded shadow"
  mode="aspectFill"
/>

```

## 部署


这次的部署也是船新版本


这个项目也算是新版 DjangoStarter 的第一次实践，所以遇到一些坑，我都写了博客记录了


* [在python项目的docker镜像里使用pdm管理依赖](https://github.com)
* [新版的Django Docker部署方案，多阶段构建、自动处理前端依赖](https://github.com)
* [使用python\-slim镜像遇到无法使用PostgreSQL的问题](https://github.com)


总得来说 daphne 服务器用着还不错。


## 小结


开头就说了，本次项目的算是比较简单的，时间主要花在前端的交互和一些细节的调整上


Taro 用来开发小程序还是丝滑的，意料中打包和发布可能遇到的问题，实际上都没有


这篇小结真的拖了太久了，我也好久没写代码了… 得抓紧时间整理思绪，然后把想做的东西都搞起来了。


