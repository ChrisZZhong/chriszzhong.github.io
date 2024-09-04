---
layout: post
title: "Regression Newman with HTML report"
date: 2024-09-04
description: "Regression Report Documentation"
tag: Microservices
---

## Regression Report with NewMan

```shell
#!/bin/bash

cd $(dirname $0)
ENV=${1:-QA}
echo "Testing $ENV environment."
[ ! -d newman ] && mkdir newman
[ -d newman/newman ] && rm newman/newman/xxx-regression-*.html

cd newman || exit
export PATH=/devops/tools/node-v12.4.0-linux-x64/bin:/devops/tools/newman-develop/bin:$PATH
newman -v

if [ $? -ne 0 ] ; then
  echo "...installing newman"
  npm config set proxy http:/
  npm config set https-proxy http:/
  npm install -g newman newman-reporter-htmlextra newman-reporter-junitfull

fi

echo "Running regression tests against $ENV"

RC=0
index=0

for file in $(find ../xxx-regression/ -name \*collection.json); do
  newman run "${file}" -e ../resources/"${ENV}".postman_environment.json -g ../xxx.json --verbose --insecure -r "cli,htmlextra,junitfull" --reporter-html-export --reporter-junitfull-export newman-junit-${index}.xml
  result=$?
  [ $result -ne 0 ] && echo ERROR && echo "ERROR: Failed to execute collection ${file}" && echo ERROR
  RC=$(expr $RC + $result)
  index=$(expr $index + 1)
done

exit "$RC"
```
