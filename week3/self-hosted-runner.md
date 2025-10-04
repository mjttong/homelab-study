# Week 3: Github Actions

## workflow 구조

```yaml
# workflow의 이름 설정
name: self-hosted runner

# workflow 트리거 이벤트 설정
# 별도의 트리거를 설정하는 대신 직접 실행하는 방식 선택
on:
  workflow_dispatch:

# self-hosted라는 이름의 jobs 설정
jobs:
  self-hosted:
    # self-hosted runner에서 실행하려 함
    runs-on: self-hosted

    # 별도의 permission은 부여하지 않음

    steps:
    # 리포지토리 코드를 runner의 작업 디렉토리로 checkout
    - uses: actions/checkout@v4
    
    - name: print main readme.md
      run: |
        echo "--------- print main readme.md ---------"
        cat README.md
```

## self-hosted runner

- 일반적으로 github actions를 실행하면 github-hosted runner를 통해 워크플로우가 실행
- 직접 개인 서버를 사용해서 self-hosted 방식으로 github actions를 실행 가능

### self-hosted runner 실습

1. github actions를 실행할 리포지토리 선택
2. Settings > Actions > Runners에서 New self-hosted runner를 통해 설정
3. 워크플로우를 실행할 runner의 OS 선택 후 github에서 제공하는 코드로 runner 다운로드 및
4. 워크플로우에 `runs-on: self-hosted` 설정
5. self-hosted runner 실행 `./run.sh`
6. `runs-on: self-hosted`가 설정된 워크플로우 실행

![Image](/week3/self-hosted-runner/idle.png)

- self-hosted runner 실행 시 runner가 idle 상태로 나타남

![Image](/week3/self-hosted-runner/active.png)

- workflow 실행 시, runner가 active 상태로 변화

![Image](/week3/self-hosted-runner/workflow.png)

- workflow 실행 성공
