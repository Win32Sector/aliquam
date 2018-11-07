---
layout: post
title: Как в Teamcity запускать другие build configurations с помощью REST API
category: article
comments: true
description: Teamcity - прекрасный инстурмент автоматизации, который предполагает, что ты в тупую запускаешь билды, состоящие из билд степов, которые идут строго подряд. В случае кастомизации, например, если вам нужно, чтобы в каком-то билд степе вызывался старт какого-то другого билда и шел параллельно, есть REST API.
tags:
    - DevOps

---

Teamcity - прекрасный инстурмент автоматизации, который предполагает, что ты в тупую запускаешь билды, состоящие из билд степов, которые идут строго подряд. В случае кастомизации, например, если вам нужно, чтобы в каком-то билд степе вызывался старт какого-то другого билда и шел параллельно, есть REST API.

Итак, чтобы cтриггерить какой-то билд, без передачи дополнительных параметров, нужно передать в тимсити xml-ку такого вида:

```
<build>
    <buildType id="buildConfID"/>
</build>
```

Если нужна кастомизация, например, передача определенной ветки, передача каких-то параметров, нужно их указать в этой xml-ке как-то так:

```
<build personal="true" branchName="logicBuildBranch">
    <buildType id="buildConfID"/>
    <agent id="3"/>
    <comment><text>build triggering comment</text></comment>
    <properties>
        <property name="env.myEnv" value="bbb"/>
    </properties>
</build>
```

Потом, нужно курлом передать эту xml в buildQueue Teamcity

```
curl -v -u user:password http://teamcity.server.url:8111/app/rest/buildQueue --request POST --header "Content-Type:application/xml" --data-binary @build.xml
```

В целом, билд степ SSH Exec, дергающий билд по его ID и передающий некоторую переменную, у меня выглядит так:

```
echo -e "<build>" > /tmp/script.xml
echo -e "    <buildType id='Build_configuration_ID'/>" >> /tmp/script.xml
echo -e "        <properties>" >> /tmp/script.xml
echo -e "            <property name='VAR1' value='some_value'/>" >> /tmp/script.xml
echo -e "        </properties>" >> /tmp/script.xml
echo -e "</build>" >> /tmp/script.xml

curl -v -u tc_service_user:tc_service_user_pass https://my-tc-server.com/app/rest/buildQueue --request POST --header "Content-Type:application/xml" --data-binary @/tmp/script.xml
```
