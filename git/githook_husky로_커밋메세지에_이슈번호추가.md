## Git hook

- Git과 관련한 어떠한 이벤트가 발생했을때 특정 스크립트를 실행할수 있도록 하는 기능을 말한다.
  - commit,merge,push시 팀 내 Git convention을 지키지 않았을 경우 작업을 중단하게 한다.
- git hook을 달아놓으면 팀원들이 원격레포 Clone시,
  - git hook에 해당하는 파일 commit-msg, pre-commit같은 파일들을 chmod +x해서 권한을 하나하나 줘야하는 번거로움이 존재하지만 팀내 규칙을 강하게 지킬수 있다는 장점이 존재한다.

## Husky

- git hook 설정을 도와주는 npm package
- 번거로운 git hook 설정이 편하다.
- package.json에 아래와 같이 설정되어 있다면
  ```tsx
  "prepare": "(husky install && chmod ug+x .husky/*)",
  ```
- pnpm install, npm install, yarn i 만 해주면 바로 git hook에 권한이 들어간다.

## 브랜치의 이슈번호를 가지고와서 커밋메세지 끝에 달기

### 사전 정보

- `feat/#1234-login_스크린_개발`
- `fix/#1234-login_스크린_개발`
- 이런 형식으로 브랜치가 구성되어 있을때 브랜치에 붙어 있는 이슈번호를 가지고와서
  - refactor: commit-msg 주석 수정 요로케만 입력해도
  - refactor: commit-msg 주석 수정 (#2465) 요로케 되게 만들어볼것 이다.

### 커밋 메세지를 날린 후 뒤에 이슈번호를 수정하는 로직을 만들어보자.

- git hook 중 사용자가 커밋메세지를 날린 후 그 메세지를 수정하는 훅은 commit-msg이다.
- 보통 .husky/commit-msg 로 파일을 추가하면 된다.
  - 이건 팀마다 설정해놓는게 다를 수 있을것 같아 각자의 상황에 맞게 하면 되겠다

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

//commit message를 관리하는 파일 -> 후술하는 사소한 궁금증들 참고
commit_msg_file="$1"
commit_msg=$(cat "$commit_msg_file")

// 브랜치 이름 가져오기
branch_name=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "HEAD")

// 브랜치 이름에서 이슈번호 정규표현식으로 추출하기 (브랜치 이름 규칙이 존재할때만 유효)
issue_number=$(echo "$branch_name" | grep -oE '#[0-9]+' | sed 's/#//')

// commit message 끝에 이슈번호 붙이고 덮어쓰기
if [ -n "$issue_number" ] && ! echo "$commit_msg" | grep -qE "#$issue_number"; then
  echo "$commit_msg (#$issue_number)" > "$commit_msg_file"

```

### 사소한 궁금증들

```bash
commit_msg_file="$1"
commit_msg=$(cat "$commit_msg_file")
```

- 왜 $1이 필요할까?
  - $1인자에는 커밋메세지가 저장된 임시 파일경로가 들어온다.
  - 스크립트가 이 파일을 수정하면서 커밋메세지가 반영된다.
  - 내가 건들인 훅은 commit-msg인데 사용자가 커밋 메세지를 작성한 후→메세지를 수정할때 시작되는 훅이기 때문에 커밋 메세지 파일이 필요하다!
- 커밋 메세지 파일은 뭘까?
  - .git/COMMIT_EDITMSG를 말한다.

> COMMIT_EDITMSG
>
> **`$GIT_DIR/COMMIT_EDITMSG`**This file contains the commit message of a commit in progress. If `git commit` exits due to an error before creating a commit, any commit message that has been provided by the user (e.g., in an editor session) will be available in this file, but will be overwritten by the next invocation of `git commit`.

[Git - git-commit Documentation](https://git-scm.com/docs/git-commit/2.2.3)

- 이 파일에는 진행중인 커밋 메세지가 포함되어있다.

## Githook 이름이 하드코딩 되어있을까?

- 문의를 받았다.
  - 기존의 commit-msg라는 파일명을 Issue number를 추가해주는 행동이 드러나게 네이밍을 하면 좋겠다는 문의였다.
- ㄸㄹㄹ 하지만 이거 git hook이라 안될텐데…! 그래서 git코드를 직접 들어가서 봤다

https://github.com/git/git/blob/master/builtin/commit.c#L1740-L1748

### hook의 동작원리

- commit hook을 실행하는 함수는 commit.c에 있다

```tsx
int run_commit_hook(int editor_is_used, const char *index_file,
		    int *invoked_hook, const char *name, ...)
{
	struct run_hooks_opt opt = RUN_HOOKS_OPT_INIT;
	va_list args;
	const char *arg;

	strvec_pushf(&opt.env, "GIT_INDEX_FILE=%s", index_file);

	/*
	 * Let the hook know that no editor will be launched.
	 */
	if (!editor_is_used)
		strvec_push(&opt.env, "GIT_EDITOR=:");

	va_start(args, name);
	while ((arg = va_arg(args, const char *)))
		strvec_push(&opt.args, arg);
	va_end(args);

	opt.invoked_hook = invoked_hook;
	return run_hooks_opt(the_repository, name, &opt);
}
```

- hook_name을 받아서 실행한다.

```tsx
if (!no_verify && run_commit_hook(use_editor, index_file, &invoked_hook,
					  "pre-commit", NULL))

	if (!no_verify &&
	    run_commit_hook(use_editor, index_file, NULL, "commit-msg",
			    git_path_commit_editmsg(), NULL)) {
		return 0;
	}
```

### 어떤 hook들이 있을까

[https://git-scm.com/docs/githooks#:~:text=are described below.-,HOOKS,-applypatch-msg](https://git-scm.com/docs/githooks#:~:text=are%20described%20below.-,HOOKS,-applypatch%2Dmsg)

- 하이퍼링크를 타고 들어가면 hook의 목록이 보인다.
- hook 목록에 있는 이름으로 파일명을 지어야만 .git이 인식한다.
- 대표적으로 많이 쓰는 것은 pre-commit, prepare-commit-msg,commit-msg같다.
  - pre-commit
    - commit을 실행하기 전에 실행
    - 이를 테면 `feat: login로직` 을 `feat : login` 로직 과 같이 작성했을때 잡아준다.
  - prepare-commit-msg
    - commit 메세지 생성하고 편집기 실행하기 전에 실행
  - commit-msg
    - commit 메세지 완성한 후 commit을 최종 완료하기 전에 실행

## 또 다른 문제, 커밋메세지가 여러줄인 경우 이슈번호가 붙지 않는다!

- 요구사항
  - 팀 내에서 git commit을 해서 여러줄의 커밋을 작성하는 일이 많았다.
  - 이를테면
    ```bash
    feat: Button 컴포넌트 리팩토링(#issue_number)
     - Button variant 속성 추가
     - iconOnly 만 받을 수 있게 type never 설졍
    ```
    - 위와 같이 커밋메세지를 날리는 경우

```tsx
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

commit_msg_file="$1"
commit_msg=$(cat "$commit_msg_file")

branch_name=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "HEAD")

issue_number=$(echo "$branch_name" | grep -oE '#[0-9]+' | sed 's/#//')

# 이슈 번호가 없으면 아무 것도 하지 않음
if [ -z "$issue_number" ]; then
  exit 0
fi

# prefix 목록
allowed_prefixes="^(feat|refactor|fix|style|docs|language|chore|delete):"

new_msg=""
first_line=true

while IFS= read -r line; do
  if echo "$line" | grep -qE '^\s*$|^\s*#'; then
    new_msg="$new_msg$line\n"
  elif [ "$first_line" = true ]; then
  # prefix 있는 경우에만 이슈 번호 붙이기
    if echo "$line" | grep -qE "$allowed_prefixes"; then
      if echo "$line" | grep -qE "#$issue_number"; then
        new_msg="$new_msg$line\n"
      else
        new_msg="$new_msg$line (#$issue_number)\n"
      fi
    else
      # prefix가 없는 경우 그대로
      new_msg="$new_msg$line\n"
    fi
    first_line=false
  else
    new_msg="$new_msg$line\n"
  fi
done < "$commit_msg_file"

# 새로운 메시지로 덮어쓰기
printf "%b" "$new_msg" > "$commit_msg_file"

```

### 예외처리를 견고하게 해보자

- 리베이스 하면 끝장나는거야

```tsx
branch_name=$(git rev-parse --abbrev-ref HEAD 2>/dev/null
```

- HEAD가 뭘까?
  - git에서 현재 작업중인 브랜치나 커밋을 가리키는 포인터이다.
  - 쉽게 말해 내가 작업중인 브랜치
- detached HEAD
  - HEAD가 브랜치를 가리키는게 아니라 특정 커밋을 가리킬때를 의미한다.
  - 흔히 리베이스나 다른 사람 커밋으로 들어가서 볼때 detached HEAD가 된다.
- git rev-parse --abbrev-ref HEAD
  - 현재 브랜치 이름만 출력해준다.
    - main, feat/#123-login 이런식으로!
  - 만약 리베이스 하는 상태라면 HEAD가 반환되는게 정상적이지만 stderr가 나는 경우가 있다.
    - 나의 경우는 그랬는데 그게 왜 그런지는 여전히 이슈 트래킹이 잘 안됐다.
  - 그런 경우 에러를 무시하기 위해 2>/dev/null를 하면 된다
  - 웬만하면 리베이스는 하지 말도록하자.. ㅠ\_ㅠ
- 2>/dev/null
  - 리눅스/유닉스에서 에러를 무시하겠다는거다.
  - 특정 명령어 실행한 후 출력이 필요없는 경우 dev/null로 리다이렉션 시키면 된다.
  - 참고로 파일디스크립터 0번은 표준입력, 1번은 표준 출력, 2번은 표준 에러이다.

[How do I get the current branch name in Git?](https://stackoverflow.com/questions/6245570/how-do-i-get-the-current-branch-name-in-git)

## Prefix로 커밋 메세지를 구분했을때 문제점 : Merge commit은?

- 이게 진짜 성가신 문제였다.
- develop 브랜치에 머지하고 나서 pull 땡기려는데 Merge commit조차 Git hook이 검사를 하는거다 (아아아ㅏㅇ아ㅏ아아앜)
- 고쳐야지 어떡해…

```bash
#!/bin/sh
. "$(dirname "$0")/../_/husky.sh"

commit_msg_file="$1"
commit_msg=$(cat "$commit_msg_file")

#NOTE: merge 커밋 메시지일 때는 검사 안 하고 빠져나가기
if grep -qE '^Merge ' "$commit_msg_file"; then
  exit 0
fi

branch_name=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "HEAD")
issue_number=$(echo "$branch_name" | grep -oE '#[0-9]+' | sed 's/#//')

# .. 이하 생략
```

- 이렇게 while문을 돌면서 커밋 메세지를 검사하기 이전에 merge commit인 경우를 확인한다.

### 테스트를 해보자

```bash

# 1. test/#2465-2 브랜치 생성 및 커밋
git checkout -b test/#2465-2
echo "This is test branch 2465-2" > test_2465_2.txt
git add test_2465_2.txt
git commit -m "feat: add file in 2465-2 (#2465)"

# 2. test/#2465-3 브랜치 생성 및 커밋
git checkout -b test/#2465-3
echo "This is test branch 2465-3" > test_2465_3.txt
git add test_2465_3.txt
git commit -m "feat: add file in 2465-3 (#2465)"

git merge --no-ff test/#2465-3
```

- 그냥 git merge test/#2465-3만 한다면 merge commit 없이 포인터만 fast-forward로 움직인다.
- git merge —no-ff를 해야 병합하면서 반드시 머지커밋을 생성할 수 있다
- . `--no-ff`는 **"포인터만 옮기지 마! 병합 흔적을 남겨!"** 라는 뜻

## appendix

[Git - Git Hooks](https://git-scm.com/book/ko/v2/Git%EB%A7%9E%EC%B6%A4-Git-Hooks)

[Git - githooks Documentation](https://git-scm.com/docs/githooks)

[[개발자로 협업하기] husky 로 git hook 적용하기](https://velog.io/@jiynn_12/%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%A1%9C-%ED%98%91%EC%97%85%ED%95%98%EA%B8%B0-husky)
