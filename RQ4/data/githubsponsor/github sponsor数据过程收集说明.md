# github sponsor数据过程收集说明

github官方的文档中提到，repository（即项目仓库）里有这样一个属性：

|​`fundingLinks` ([`[FundingLink!]!`](https://docs.github.com/en/graphql/reference/objects#fundinglink))|The funding links for this repository.|
| -----------| ----------------------------------------|

这个属性表示了该项目开通的受捐赠的方式。

fundingLinks这个属性进一步划分又包含了platform和url这两个属性

对于platform，这个属性，官方文档中的描述如下：

|​`platform` ([`FundingPlatform!`](https://docs.github.com/en/graphql/reference/enums#fundingplatform))|The funding platform this link is for.|
| -----------| ----------------------------------------|

这个属性表明了该项目在哪些赞助平台上受到捐助。该属性目前的取值是固定的一些枚举值，官方文档罗列了相关的具体平台。

> The possible funding platforms for repository funding links.
>
> #### Values for `FundingPlatform`​
>
> **BUY_ME_A_COFFEE**
>
> Buy Me a Coffee funding platform.
>
> **COMMUNITY_BRIDGE**
>
> Community Bridge funding platform.
>
> **CUSTOM**
>
> Custom funding platform.
>
> **GITHUB**
>
> GitHub funding platform.
>
> **ISSUEHUNT**
>
> IssueHunt funding platform.
>
> **KO_FI**
>
> Ko-fi funding platform.
>
> **LFX_CROWDFUNDING**
>
> LFX Crowdfunding funding platform.
>
> **LIBERAPAY**
>
> Liberapay funding platform.
>
> **OPEN_COLLECTIVE**
>
> Open Collective funding platform.
>
> **PATREON**
>
> Patreon funding platform.
>
> **POLAR**
>
> Polar funding platform.
>
> **THANKS_DEV**
>
> thanks.dev funding platform.
>
> **TIDELIFT**
>
> Tidelift funding platform.

这里有一个取值是GITHUB，我们知道github sponsor是依附于github的一个赞助平台，并不是完全独立的平台，可以认为这里的取值GITHUB其实就是对应github sponsor这种赞助方式。

因此我们可以通过官方的graphql接口来爬取repository信息并查询repository的fundingLinks中是否含有GITHUB, 如果有，表明该项目是受到github赞助的，也就是符合我们要求的项目， 接着通过graphql的接口爬取这些项目的基本信息即可。

相关代码如下（仅截取部分主要代码）：

```python
...
	GRAPHQL_QUERY = """
	query ($owner: String!, $repo: String!) {
	  repository(owner: $owner, name: $repo) {
	    nameWithOwner
	    fundingLinks {
	      platform
	      url
	    }
	  }
	}
	"""
...
	variables = {"owner": owner, "repo": repo}
    payload = {'query': GRAPHQL_QUERY, 'variables': variables}
	response = requests.post(
        'https://api.github.com/graphql',
         headers=headers,
         json=payload,
         timeout=10
    )
...
	data = response.json()
...
   repo_data = data['data']['repository']
   funding_links = repo_data.get('fundingLinks', [])

   # 检查是否有GITHUB平台
   for link in funding_links:
       if link['platform'] == 'GITHUB':
           return True
```
