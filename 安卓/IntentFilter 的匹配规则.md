## IntentFilter 的匹配规则 

IntentFilter 中的过滤信息有 action、category、data，他们是在四大组件的标签下进行配置的，需要同时匹配列表中的 action、category 以及 data 才算匹配成功，否则失败。一个过滤列表中可以有多个 action、 category、 data，**一个 Activity 可以有多个匹配规则列表**，一个 Intent 只要能够匹配任何一组就可以成功启动该 Activity 了，也就是匹配成功了。 

action 匹配规则： action 的匹配规则要求 Intent 中的 action 必须存在，并且必须和过滤规则中的其中 一个 action 相同，他的匹配是区分大小写的，在代码中通过 setAction 进行设置。

category 匹配规则： category 匹配要求** Intent 中可以没有 category，但是只要有 category，不管你有几 个，每一个都要和过滤规则中的其中之一 category 匹配**，代码中通过 addCategory 进行设 置；那么为什么 Intent 可以没有 category 而必须有 action 呢？原因在于**系统在调用 startActivity 或者startActivityForResult的时候默认会为 Intent 添加上"android.intent.category.DEFAULT"这个 category，所以如果我们没有为 Intent 设置 category，默认他会匹配包含有 "android.intent.category.DEFAULT"的 Activity 的。

data 匹配规则： data 是由**mimeType 和 URI** 组成，mineType 表示媒体类型，URI 的默认值为 content 和 file，也就是说**如果你的 Intent 没有指定 URI 部分， 那么默认情况下 URI 的 schema 部分是 content 或者 file **的，data 的匹配规则和 action 类似，也就要求 Intent 中必须包含 data 数据，并且 data 数据可以完全匹配过滤规则中的某一个 data。



另外使用隐式Intent启动Activity的时候可以做一下判断规避错误，确定Activity是否匹配我们的隐式Intent，可以采用**PackageManager的resolveActivity方法或者Intent的resolveActivity方法**，如果找不到匹配的Activity就会返回null，以及PackageManager的queryIntentActivity方法会返回所有成功匹配的Activity信息。**不含有DEFAULT这个category的Activity是无法接收隐式Intent的**。

