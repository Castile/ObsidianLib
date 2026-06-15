
参考NiFi官方的releaseNote： https://cwiki.apache.org/confluence/display/NIFI/Release+Notes#ReleaseNotes-Version1.27.0

分析一下从1.26版本到2.2.0版本的不兼容点

## 1.27.0
- [[NIFI-13415](https://issues.apache.org/jira/browse/NIFI-13415)] - Deprecate Property Protection and Encrypt Config 
过期敏感属性加解密的工具和api， 将在2.2.0版本进行删除， 相关issue可以参考：
https://issues.apache.org/jira/browse/NIFI-13414

翻译大意：
NiFi和NiFi Registry支持几种策略来加密和解密位于NiFi中的应用程序属性。属性除了保护flow配置中的敏感值之外。最初的实现使用AES-GCM支持属性加密，密钥位于bootstrap.conf中。随后的实现提供了与外部秘密管理服务的集成。支持这些实现需要大量的第三方库，并且不提供可扩展实现的公共方法。这些现有方法的安全性和可维护性都存在问题，因此必须弃用它们，并从当前的主分支中删除它们。
从安装的整体角度来看，本地AES-GCM实现不能提供足够的安全性。虽然值在nifi。属性可以加密，加密密钥必须以明文形式存储在bootstrap.conf中，这两个文件都位于conf目录下。任何具有读取文件系统权限的操作系统用户都可以将这些配置放在一起读取nifi.properties中的值。
基于服务的实现使用需要与服务交互才能读取的属性值引用或加密值来提供外部化。这种方法是有益的，但它需要为每个服务提供者维护单独的实现，并且还需要在补充引导配置文件中配置访问凭据。这些基于服务的实现有很大的依赖树，每个依赖树的内容都存储在lib目录下的properties目录中。为所有受支持的实现合并服务提供者库的副本大大增加了标准发行版的重量，并且由于缺乏依赖隔离，使其更难以维护。
现有的nif -property-protection-api和提供的实现不支持应用程序属性安全的可维护模式。nif -toolkit-encrypt-config模块还包含大量用于加密应用程序属性的带外运行所需的代码。encrypt-config命令是与标准NiFi发行版分开打包的，这使得它在常见部署场景中用处不大。
综合这些问题，现有的nifi property保护模块。属性应该从主分支中移除。这将在短期内提供一个简化的分发，并且还为考虑不受相同类型的安全问题影响的更健壮的方法提供更好的基础。


## 1.28.0
新特性：
- Add performance tracking to ProcessGroupStatus

其他的没有什么不太兼容的点


## 2.0.0
这是第一个nifi2.x版本。这个版本进行了大量的issue解决和不兼容变更
过时的组件和特性可以查看列表：
https://cwiki.apache.org/confluence/display/NIFI/Deprecated+Components+and+Features
过时的组件一律不允许使用，需要在界面中删除。

迁移指导： https://cwiki.apache.org/confluence/display/NIFI/Migrating+Deprecated+Components+and+Features+for+2.0.0

- Moved several modules to optional build profiles, requiring separate NAR download from the Central Repository
    - nifi-hadoop-nar
    - nifi-hadoop-libraries-nar
    - nifi-hbase-nar
    - nifi-hbase2_client-service-nar
    - nifi-kudu-nar
    - nifi-parquet-nar
    - nifi-solr-nar

**Notable Upgrades**

- Spring Framework 6
- Jetty 12
- Jakarta Servlet API 6
- Jakarta XML Binding 4
- Swagger 2 annotations
- OpenAPI 3.0 REST API specification

- Relocated JoltTransformJSON and JoltTransformRecord from nifi-standard-nar to nifi-jolt-nar
  - Moved from [SimpleDateFormat](https://docs.oracle.com/en/java/javase/21/docs//api/java.base/java/text/SimpleDateFormat.html) to [DateTimeFormatter](https://docs.oracle.com/en/java/javase/21/docs/api//java.base/java/time/format/DateTimeFormatter.html) for date and time parsing and formatting
- Moving from Servlet API 3 and Servlet API 6 requires custom user interface extensions to be updated and recompiled

nifi-api独立仓库
新增rest api允许上传资产和自定义nar budnles了
- [[NIFI-13489](https://issues.apache.org/jira/browse/NIFI-13489)] - Remove Kafka 2.6 Processors


