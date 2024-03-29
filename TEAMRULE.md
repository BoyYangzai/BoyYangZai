### Why 🤔
   之前在字节的时候 看过剪映团队ld 曾琛 写的 剪映 性能优化，超脱于市面上所有的专栏，确实很佩服，是真正的沉淀出了经验，故在此沉淀一下我的团队和业务经验<br>
   一部分是学习来自于开源社区巨佬们，另一部分是对自己待过的四个厂百度、支付宝、字节、腾讯，自己觉得最舒服的 TEAMWORK 和 Code Style 沉淀

### TEAM WORK
 - 业务中尽量避免使用 //@ts-ignore //@ts-noCheck，过不去的 Lint，要么是技术不到位要么是懒 
 - ESlint Prettier 与 VScode Workspace<br>
这里是在腾讯的时候，发现的一个工作流问题，有一些 Pipe Line 会阻止 warning 的lint error，但是我本地没有 Eslint，只有 Prettier，导致无法在开发阶段去发现并且自动 Ctrl+S 去修复<br>
最佳实践：ESlint 在开发阶段自动读取 WorkSpace 中的配置并且 Ctrl+S 会自动去 fix 这些 warning<br>
No Prettier ❌
```json
// .vscode/settings.json
{
  "eslint.validate": [
    "javascript",
    "typescript",
    "javascriptreact",
    "typescriptreact",
    "vue",
    "json",
    "jsonc",
    "json5"
  ],
  "stylelint.validate": [
    "css",
    "less",
    "scss"
  ],
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  }
}
```
   - 非开源仓库请务必 lock，防止某天 Pipe Line 构建完成后，线上崩溃，以及被投毒<br>
   - 官网开发首选 amis，参考 Antv 官网、AntDesign 官网、CapCut 官网等 (首先自己公司的框架，我一般用 Dumi)<br>
     优点：<br>
     1.后续迭代便利，开发难度小<br>
     2.只需修改 Config 即可更换网站信息<br>
     3.静态站点的 SSG<br>
     4.有问题随时可以找 [@Peach老师](https://github.com/PeachScript)(🤪
   - wasm 不是必须品，应用场景较少，主要用于：<br>
     1.大规模计算，视频、绘制、引擎 如 Antv 可视化渲染、视频团队、抖音烟花特效等<br>
     2.突破原有 SDK 的性能瓶颈 如 字节电商 IM SDK、监控 SDK<br>
     3.增加被反编译的成本 如 字节风控 SDK<br>
   - umi3 升 4 比较容易，3 已经不维护,不会 onCall 和 新 Feature，但市面上很多团队还在使用<br>
     收益：<br>
     1.构建速度 以及 Hot Module update 显著提高<br>
     2.新特性以及可以及时被响应的 Github issue<br>
     3.毕竟咱也是 umi 团队成员，基本不会有无法解决的问题<br>
     缺点：需要一定的人力投入 以及 全量回归 成本<br>
   - 巨型 Monorepo 仓库不同的 Pkg 下的依赖版本冲突 以及 老仓库巨石应用 某个基层包 的升级 都可以通过 Resolution 去解决
   - 大厂以及开源项目的国际化：<br>
     原理：其实都是 react-i18n，比较简单的去取对应 lang key 下的 value<br>
     基建团队做了什么？<br>
     答：参考 Element-plus 的国际化，通过一个第三方的翻译平台，方便翻译人员操作，同时生成对应的 JSON 文件，只不过大厂不是第三方的服务，而是有自己的国际化翻译平台<br>
     所以如果未来去初创/中小公司带团队，海外业务国际化的技术选型可以使用第三方的服务
   - 用户行为分析的埋点一般是 PM 提需求，在哪里埋，数据分析需要什么指标，前端埋点比较简单，一般只需要带上对应的 Event key value
   - 性能监控指标 C 端业务会看重这个 一般 X-FMP FCP 会比较重要一点
   - 大量动态加载的静态资源，图片、gif、视频、字体等，都可以通过 InDexDB 去做一层 Cache，有效提高请求速度
   - 合理使用 preconnect、prefetch、preload
### Code Style
#### Hooks<br>
- 与 React 原生 Hook 写法保持一致 小驼峰，返回 State 与 UpdateState
```tsx
    const useCpData = () => {
      ...
      return [cpData, setCpData]
    }
```

  -  Hook 与 Modal State
在一些情况下 在 Hook 中处理 Modal 中的状态去复用极为方便
举个例子 有大大小小 10 个模块，且可以单独通过 url 进入，此时 Modal 中不一定存在对应的 State Value,但每个页面都需要这个 Modal State<br>
> 优点:
>    - 只负责 Save Init State Or Get State from Modal，脱离业务逻辑
>    - 逻辑复用极其简单，无需在每个模块去 Init State
```tsx
    const useUserInfo = (props) => {
      const userInfo = useRedux(state: GlobalModal => state.Modal.userInfo)
      const setUserInfo = useRedux(state: GlobalModal => state.Modal.setUserInfo)

      const fetchUserInfo = () => {
          dispatch(params) // saveState to Modal
       }

      useEffect(() => {
      if(!userInfo){
        fetchUserInfo()
        }
      },[])
      ...
      return [userInfo, setUserInfo]
    }
```
  - 功能型 Hook 不与业务数据相关联，请提取出 @TEAM/utils 第三方 PKG
    如 useEventOutSide 这种

 Tips: Meta 团队提供了非常多的 Hooks，但是很多 Hooks 都是为 SSR 服务的，一般的业务开发都无需关心   
#### Component<br>
  - 组件命名 - 大驼峰 文件名：大驼峰 or 连接符
  - 业务组件 - 推荐 Data(数据) 与 View(视图) 分离,用相应的 Data Hook 去 管理/创建 CP 单例<br>
    举个例子，这里有一个商业化复杂应用场景下的 Table Modal, 虽然复杂 但是极其耐用 <br> 
    具体的功能大概是：<br>
    复杂业务场景的 Search + 后端分页 + 复杂业务表格数据以及嵌套多重数据 + 自定义 UI 插槽 + 以及各种 Modal Render Props
```tsx
  // 通用组件 非 Page CP
  // components/TableModal/index.tsx
  interface TableModalProps{
    records: TableRecords;
    columns: TableColumns;
    pagination: PaginationInfo;
    searchProps: TeamSearchCpProps,
    ...
  }

  const TableModal:FC = () => {
    return <> ... </>
  }

  export default TableModal;
  // components/TableModal/hooks/useTableModalData.ts
  const useTableModalData = () => {
    const [data, setData] = useState<TableModalData>([])
    const [pagination, setPagination] = useState<PaginationInfo>({ pageIndex: 1, pageSize });
    const [tableStatus, setTableStatus] = useState<TableStatus>(TableStatus.empty)
    ... Hook with Data 
    return [data, pagination, tableStatus, setData, setPagination, setTableStatus]
  }
```

#### Modal State
   - 去不同公司不同团队写 React，必须要适应的是不同的`状态管理`<br>
     基于 Redux 的 Dva、easy-xxx 等，Mobx、zustand、jotai等<br>
     我本人在过的团队使用的基本上都是围绕 redux 理念的生态，但是现在有不少团队在使用其他的状态管理工具<br>
   - 以页面为维度去划分 Namespace 其实是有点混乱的，如Dva，本质上还是全局的状态
     个人推荐 全局状态划分不同的 Modal，易于区分不同的数据模型、清晰易懂、容易定位
#### 模块
   - 导入顺序优先级<br>
     1.第三方 Pkg<br>
     2.本地 JS 模块<br>
     3.非 JS 模块<br>
   - 类型导入最好带上 type 关键字
   - 类型声明放在 ./type.ts
> 正确示例✅
```ts
import { Cp1, Cp2, ... } from 'antd';
import { xxx } from 'umi;

import MyComponent from './MyComponent';
import useHook from './hooks/useHook';

import type { myType} from './type.ts';
import styles from './index.less'; 
 ```
#### Code
   - 避免频繁使用 useMemo、useCallback
   - 依赖超过 5 个，去掉 Hook，避免增加心智负担
   - 请将复杂业务/计算逻辑 统一抽离放在 useMemo 中
```ts
   const mergedXXX = useMemo(()=>{
      const XXX = ...
      return conetext.XXX ?? XXX
   },[dep1, dep2, ...)
```
   - 避免三元表达式嵌套
   - 注意空格，养成习惯
   - 在公司代码中，尽量使用 if 条件语句替代可扩展的可选链逻辑 ?
> 正确 ✅
```ts
   interface UserInfo {
      [key: string]: string
   }
   const res: {
      userInfo: UserInfo
   } = dispatch();

   if (res?.useInfo) {
     console.log(res.useInfo.age);
   }
```
   - 调用不一定存在的函数，请记得 fn?.()，一般用于组件封装场景，以及数据通信

