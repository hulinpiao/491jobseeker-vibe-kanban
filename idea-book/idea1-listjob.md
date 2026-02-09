
我需要功能模块儿是：在本地localhost上显示数据库里的job 信息

我本地的mongodb （mongodb://localhost:27017/491jobseeker) 的 unified_jobs database 里已经有2026_02_07的collection。我需要前后端去吧这个collection里的所有数据展示出来。 \                                                                                
  功能：                                                                 
  1。 需要有search keyword/location 功能。 \\         
  2。 list jobs。 默认按照日期排序。 \ 
  3。 listed jobs 只显示 job title，和部分job detail 内容。 \                 
  4。 点击job，需要跳转到job detail 页面。 \                                                                         

结合当前v5里的的cc agent teams 和vibe kanban。 我的想法是： 

首先是功能开发前的市场调研，企划和架构设计。
请择机选择superpowers skill 去分析。使用苏格拉底式分析法去分析feature request 并且随时给我提问去分析核心功能。 
  
  并在/Users/hulin/Documents/projects/491jobseeker/v5/modules/vibe-kanban里生成PRD， solution architecture，以及 task plan文档。   
 -  solution architecture 需要符合当前codebase（v5） 里的 架构和要求。 
  - task plan 文档需要创建最少的Epic， 每个Epic下需要创建tasks。 每个task需要有详细的操作md文档（包括标签来说明需要在那个repo进行开发 和详细的task plan）。 以便后面上传vibe tickets。 
  - 最后，以每个epic为一个ticket。并且在ticket 下附加所有在epic下的task md文档。 ticket descrption 需要说明这个epic 简要和成功指标，还有可能所需要的skills（比如test agent 可能需要E2E test skill，前段agent 根据架构可能需要react或者nextjs skill 等等） 还有，以什么顺序去执行eipc里的tasks。 

注意， 我需要用TDD 方法去进行开发。 开发阶段我会用到agent teams， 让team lead agent 帮我生成多个agents 去分工开发。

等上面企划工作完成，我需要我人工检查vibe kanban里的tickets。 然后 我们需要创建以下 cc agent teams（开发组）： 
1. team lead ： 这是orchestrator agent。也需要扮演project management角色。 他可以调度多个agents去做不同的任务。 并且通过vibe kanban MCP 去实施监控和更新kanban里的每个ticket的状态。 根据状态的不同，可以调度agent去执行ticket里的内容。比如： 当一个ticket 需要开始时， team lead 需要根据ticket 的功能需要去调度不同的agent 去执行某一个具体的任务。  
2. Frontend： 这是前段 agent。 他需要执行所有前段开发任务，包括UI设计。 以及进行前段的 link+typecheck(eslint/typescript) unit test等。 任务工作repo是/Users/hulin/Documents/projects/491jobseeker/v5/modules/frontend
3. Backend： 这是后端agent， 他需要执行所有的后端开发和数据库开发任务。 和api集成测试。 任务工作repo是/Users/hulin/Documents/projects/491jobseeker/v5/modules/backend
4. Test： 进行e2e 和功能测试，和code review。确保服务按照期望的结果正常运行。任务工作repo是/Users/hulin/Documents/projects/491jobseeker/v5/modules/test。
5. 注意， 每个agent 在repo里执行某个epic时必须要创建一个feature branch（从main里拉去） 然后进行开发。 等Test agent 说成功， team lead 需要把ticket 移动到complete。 然后raise 一个PR。 

vibe kanban： 
todo： 为开始的任务。企划组需要在这里开启ticket， 开发组的team lead 得到通知说这里有新ticket， 根据ticket里的内容安排给不同的agent去执行任务。 
in process： team lead在开始任务前需要把ticket 移动到这里。并且根据任务安排frontend或者backend agent去工作。 当任务结束后每个agent 需要告诉team lead， 任务结束。team lead 会把ticket 移动到下一个阶段（in review）。 
in review： 当ticket 到这个阶段， test agent 需要工作。 如果所有测试通过， test agent 需要告诉 team lead。然后team lead 把ticket 放到done。 如果测试没有通过。需要告诉我team lead。并且team lead 要放回in process 并且安排agent 去修复。 
done： 完成的任务。 
