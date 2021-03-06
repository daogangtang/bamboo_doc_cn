# 模型custom API

Redis相对于传统的SQL数据库，一个很大的优势就是灵活性。为了能利用这种灵活性，Bamboo设计了一套完整的的自定义k-v存储接口，用户可以方便地使用它们。它们特别适用于记录一些模型相关的属性到数据库中。 

这套API可以被模型调用，也可以被实例调用。

## APIs
##### model_obj:setCustom(key, val, stype, scores)	

	创建一个custom key，将val值写入此key中。
	- key		自定义的key
	- val		要存的值，可以为string, list, table, 
	- stype		存储类型。只能取nil, 'string', 'list', 'set', 'zset', 'hash' 中的一个
	- scores	当存储类型为'zset'的时候，可以为zset的元素指定score，是一个数字list

##### model_obj:getCustom(key, stype)	

	获取custom key的所有内容
	
	- key		自定义的key
	- stype		指明要取的键的类型，取'string', 'list', 'set', 'zset', 'hash' 中的一个

##### model_obj:getCustomKey(key)

	获取redis中真实的custom key
	
	- key		自定义的key

##### model_obj:delCustom(key)	

	删除此custom key
	
##### model_obj:updateCustom(key, val)	

	将val值更新到custom key中去，注意，是覆盖关系
	- key		与setCustom意义相同
	- val		与setCustom意义相同

##### model_obj:removeCustomMember(key, val)	

	删除custom key中的val元素
	- val		某一个元素

##### model_obj:addCustomMember(key, val, stype, score)	

	添加一个member到custom key中去
	- val		新元素值
	- stype		指定键key的类型
	- score		可选。当st为'zset'的时候，可以为其赋予一个score值，以决定添加后的顺序

##### model_obj:hasCustomMember(key, mem)

	判断custom key中，是否有成员 mem。
	- mem		某一个元素值。一般为数字或字符串

##### model_obj:numCustom(key)	

	测量custom key中有几个元素

## 说明
在内部，custom key是被限制在`model_obj`下面的。也就是说，不存在独立的custom key，总是需要依附某一个model而存在。

比如，User模型，使用  

	User:setCustom('test', 'have a test')  

后，在redis中存储的key是 `User:custom:test`, 值为 'have a test' 。

再比如，user1对象，是User的一个实例，调用

	user1:setCustom('test', 'i am an instance.')
	
，在redis中存储的key是 `User:1:custom:test`，值为 'i am an instance.'。

如果实在找不到要用到的custom key与哪一个模型有关联，就用`Model`模型。

执行  

	Model:setCustom('test', 'have a test')  

后，在redis中存储的key是 `Model:custom:test`, 值为 'have a test' 。 


## Note

1. 尽管custom支持string, list, set, zset, hash五种存储结构，但每种结构最基本的单元还是一个字符串，在设计的时候要注意；
2. 尽量不要使用纯数字作为key参数，名字尽量取得有意义；
3. 将custom key限制在某一个模型名字空间下面，是为了防止滥用custom key。
