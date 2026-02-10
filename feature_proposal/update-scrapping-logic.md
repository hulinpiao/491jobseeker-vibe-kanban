先分析当前的brightdata codebase。 我想把当前的爬虫和ELT 逻辑更改一下。 
1. 想每天下午1点 brightdata 分别从indeed， seek， linkedin爬取以下条件的工作：
    job keywords： Devops enginner， SRE， Cloud engineer， platform enineer， senior devops
    location： remote， onsite from canberra or brisban and sydney周围的偏远地区
    posted by： 24hours/1天前

2. 把从indeed， linkedin， seek 爬取过来的job 数据分别报管到（job_scraper下的各自平台的collection下： 目前有indeed_jobs, seek_jobs, linkedin_jobs collection） 

3. 从各自平台collection 里抓取今天创建的item 进行归一化。通过llm（perfer GLM）去做归一化就好了。 需要分析当前indeed_jobs, seek_jobs, linkedin_jobs collections的data structure，然后给出最终归一化的数据架构。 然后把归一化的数据以今天日期为命创建一个collection在normalization database 下面。 

4. 再从normalization 数据库里找到今天日期的数据库， 利用LL（GLM） 去进行jd的分析。 主要分析： JD 里是否有citizenship，PR等身份要求和 NV1， NV2， baseline 等security clearance 的硬性要求这些是491签证者无法申请的一个硬性要求。 JD里要求的skill 是否匹配我的resume里的exp 和skill （这部分需要跟你具体分析） 。 最终把符合我要求的job 过滤到一个独立的数据库里（创建新database， 按照日期去创建最终版的collection）。 

5. 统计数据： 需要一个database 去保管我们爬虫的数据和 491和skill match 筛选前和后的数据统计（比如有几个job是因为身份被过滤， 几个是因为skill/exp 不符合而被过滤） 
