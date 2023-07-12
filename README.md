# fabric_policy_analays
根据通道配置块分析fabric 2.5.0 policy

**hyperledger fabric一直在努力发展去中心化，随着版本的更迭，之前一些中心化的设计慢慢被移除kafkaraft,consortium,genesis channel...**

fabric 一直以来因为繁琐的配置项被人诟病,tsc一直在想办法简化fabric的配置同时保证丰富的功能,所以从2020年开始系统通道的删除就被提上了日程,直到Jason Yellick大神提交了osnadmin CLI pr https://github.com/hyperledger/fabric/pull/2077

fabric 2.3版本首次开始尝试移除genesis channel,使用osnadmin工具代替系统通道管理raft集群,在移除过程中连同删除了多余配置项consortium这个逻辑概念,当然这并不是强制性的移除用户可以通过配置项,决定自己的raft集群管理方式,使用系统通道,还是使用osnadmin工具.至于osnadmin相比于系统通道的优劣,可以移步去hyperledger wiki查看osnadmin的设计理念

fabric 2.5版本宣布kafkaraft 将处于deprecated状态,后续版本将会移除

伴随着consortiums从通道配置中删除,fabric的policy结构发生了一些变化,策略的层级变得相对来说比较简单

fabric 2.5.3因为json格式不支持注释这里转为yaml
```yaml
# 观察fabric的通道配置块,可以看到在fabric的配置树中,每一层通道及级配置都存在三个必要字段version,mod_policy,policys
# 同时fabric通过对子策略的引用实现的隐式元策略,例如下面channel_group的polices配置中的Admins策略是通过更低级的也就是channel_group.groups.Applications.polices中的Admins配置实现的,以此类推在配置树的基础上实现了策略树
channel_group:
  groups:
    Applications:
      managerMSP:
        groups: {}
        mod_policy: Admins
        policies:
            Admins:
              mod_policy: Admins
              policy:
                type: 1
                value:
                  identities:
                    - principal:
                        msp_identifier: managerMSP
                        role: ADMIN
                      principal_classification: ROLE
                  rule:
                    n_out_of:
                      "n": 1
                      rules:
                        - signed_by: 0
                  version: 0
              version: "0"
        values: "***"
        version: "1"
  mod_policy: Admins
  values:
  policies:
    Admins:
      mod_policy: Admins
      policy:
        type: 3
        value:
          rule: ANY
          sub_policy: Admins
      version: "0"
  version: "0"
```