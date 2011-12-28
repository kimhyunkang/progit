# Customizing Git #

지금까지 어떻게 Git이 동작하고 어떻게 Git을 사용하는지 설명했다. 이제 Git을 좀 더 쉽고 편하게 사용할 수 있도록 만들어 주는 도구를 살펴볼 것이다. 이 장에서는 먼저 많이 쓰이는 설정 그리고 훅 시스템을 먼저 설명한다. 그 후에 Git을 한 번 Customize해 볼 것이다. 자신의 프로젝트에 맞게 Git을 사용하기 편하게 해보자.

So far, I’ve covered the basics of how Git works and how to use it, and I’ve introduced a number of tools that Git provides to help you use it easily and efficiently. In this chapter, I’ll go through some operations that you can use to make Git operate in a more customized fashion by introducing several important configuration settings and the hooks system. With these tools, it’s easy to get Git to work exactly the way you, your company, or your group needs it to.

## Git 설정하기 ##

1장에서 설명했지만, 제일 먼저 해야 하는 것은 `git config` 명령으로 이름과 e-mail 주소를 설정하는 것이다:

As you briefly saw in the Chapter 1, you can specify Git configuration settings with the `git config` command. One of the first things you did was set up your name and e-mail address:

	$ git config --global user.name "John Doe"
	$ git config --global user.email johndoe@example.com

이렇게 설정하는 몇 가지 중요한 것을 더 배우게 될 것이다.

Now you’ll learn a few of the more interesting options that you can set in this manner to customize your Git usage.

아주 기초적인 설정은 1장에서도 설명했지만, 이번 장에서 다시 한 번 복습한다. Git은 내장된 기본 규칙을 따르지만, 설정 파일에 들어 있는 것이 있다면 그에 따른다. Git은 먼저 `/etc/gitconfig` 파일을 찾는다. 이 파일은 해당 시스템에 있는 모든 사용자와 저장소에 적용되는 설정 파일이다. `git config` 명령에 `--system` 옵션을 주면 이 파일을 사용한다.

You saw some simple Git configuration details in the first chapter, but I’ll go over them again quickly here. Git uses a series of configuration files to determine non-default behavior that you may want. The first place Git looks for these values is in an `/etc/gitconfig` file, which contains values for every user on the system and all of their repositories. If you pass the option `--system` to `git config`, it reads and writes from this file specifically. 

다음으로 `~/.gitconfig` 파일을 찾는다. 이 파일은 해당 사용자에게만 적용되는 설정 파일이다. `--global` 옵션을 주면 Git은 이 파일을 사용한다.

The next place Git looks is the `~/.gitconfig` file, which is specific to each user. You can make Git read and write to this file by passing the `--global` option. 

마지막으로 현재 작업 중인 저장소의 Git 디렉토리에 있는 `.git/config` 파일을 찾는다. 이 파일은 해당 저장소에만 적용된다. 각 설정 파일에 중복된 설정이 있으면 설명한 순서대로 덮어쓴다. 예를 들어 `.git/config`와 `/etc/gitconfig`에 같은 설정이 들어 있다면 `.git/config`에 있는 설정을 사용한다. 설정 파일은 손으로 직접 편집해도 되지만 보통 `git config` 명령을 사용하는 것이 더 편하다.

Finally, Git looks for configuration values in the config file in the Git directory (`.git/config`) of whatever repository you’re currently using. These values are specific to that single repository. Each level overwrites values in the previous level, so values in `.git/config` trump those in `/etc/gitconfig`, for instance. You can also set these values by manually editing the file and inserting the correct syntax, but it’s generally easier to run the `git config` command.

### 클라이언트 설정 ###

설정은 클라이언트와 서버로 나누어 볼 수 있다. 대부분은 개인작업 환경과 관련된 클라이언트 설정이다. Git에는 설정거리가 매우 많은데, 여기서는 Workflow를 관리하는 데 필요한 것과 잘 사용하는 것만 설명한다. 한 번도 겪어 보지 못할 상황에서나 유용한 옵션까지 포함하면 설정거리가 너무 많다. Git 버전마다 옵션이 조금씩 다른데, 다음과 같이 실행하면 설치한 버전에서 사용할 수 있는 옵션을 모두 볼 수 있다.

The configuration options recognized by Git fall into two categories: client side and server side. The majority of the options are client side—configuring your personal working preferences. Although tons of options are available, I’ll only cover the few that either are commonly used or can significantly affect your workflow. Many options are useful only in edge cases that I won’t go over here. If you want to see a list of all the options your version of Git recognizes, you can run

	$ git config --help

어떤 옵션을 사용할 수 있는지 `git config`의 매뉴얼 페이지에 자세히 설명돼 있다.

The manual page for `git config` lists all the available options in quite a bit of detail.

#### core.editor ####

Git은 편집기를 설정하지 않았거나 설정된 편집기를 찾을 수 없으면 Vi를 실행한다. 커밋할 때나 tag 메시지를 편집할 때 설정된 편집기를 실행시켜 준다. `code.editor` 설정으로 편집기를 설정한다:

By default, Git uses whatever you’ve set as your default text editor or else falls back to the Vi editor to create and edit your commit and tag messages. To change that default to something else, you can use the `core.editor` setting:

	$ git config --global core.editor emacs

이렇게 설정하면 메시지를 편집할 때 환경변수에 설정된 편집기 말고 항상 Emacs를 실행해준다.

Now, no matter what is set as your default shell editor variable, Git will fire up Emacs to edit messages.

#### commit.template ####

커밋할 때 Git이 보여주는 커밋 메시지는 이 옵션에 설정한 템플릿 파일이다. 예를 들어 `$HOME/.gitmessage.txt` 파일을 다음과 같이 만든다:

If you set this to the path of a file on your system, Git will use that file as the default message when you commit. For instance, suppose you create a template file at `$HOME/.gitmessage.txt` that looks like this:

	subject line

	what happened

	[ticket: X]

이 파일을 `commit.template`에 설정하면 Git은 `git commit` 명령이 실행해주는 편집기에 이 메시지를 기본으로 넣어준다.

To tell Git to use it as the default message that appears in your editor when you run `git commit`, set the `commit.template` configuration value:

	$ git config --global commit.template $HOME/.gitmessage.txt
	$ git commit

그러면 commit할 때 자동으로 편집기에 다음과 같은 메시지를 채워준다:

Then, your editor will open to something like this for your placeholder commit message when you commit:

	subject line

	what happened

	[ticket: X]
	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	# modified:   lib/test.rb
	#
	~
	~
	".git/COMMIT_EDITMSG" 14L, 297C

소속 팀에서 커밋 메시지 규칙을 정하면 그 규칙을 템플릿 파일로 만든다. 그리고 Git이 그 파일을 사용하도록 설정하면 규칙을 좀 더 쉽게 따를 수 있다.

If you have a commit-message policy in place, then putting a template for that policy on your system and configuring Git to use it by default can help increase the chance of that policy being followed regularly.

#### core.pager ####

Git은 `log`나 `diff`같은 명령의 메시지를 출력할 때 페이지로 나누어 보여준다. 기본으로 사용하는 명령은 `less`이나 `more`를 더 좋아하면 `more`라고 설정한다. 혹은 페이지를 나누고 싶지 않으면 빈 문자열로 설정한다:

The core.pager setting determines what pager is used when Git pages output such as `log` and `diff`. You can set it to `more` or to your favorite pager (by default, it’s `less`), or you can turn it off by setting it to a blank string:

	$ git config --global core.pager ''

이제 명령을 실행하면 Git은 길든지 짧든지 결과를 한 번에 다 보여 준다.

If you run that, Git will page the entire output of all commands, no matter how long they are.

#### user.signingkey ####

2장에서 설명했던 Annotated Tag를 만들 때 사용하는 GPG 키를 설정해 둘 수 있다. GPG 키를 다음과 같이 설정해 두면 서명할 때 편하다:

If you’re making signed annotated tags (as discussed in Chapter 2), setting your GPG signing key as a configuration setting makes things easier. Set your key ID like so:

	$ git config --global user.signingkey <gpg-key-id>

`git tag` 명령을 실행할 때 굳이 키를 명시하지 않고도 서명할 수 있다:

Now, you can sign tags without having to specify your key every time with the `git tag` command:

	$ git tag -s <tag-name>

#### core.excludesfile ####

Git이 무시하는 untracked 파일은 `.gitignore`에 해당 패턴을 적으면 된다고 2장에서 설명했다. 해당 패턴의 파일은 `git add` 명령으로 추가해도 Stage되지 않는다. `.gitignore` 파일을 저장소 밖에 두고 관리하고 싶으면 `core.excludesfile`에 해당 파일의 경로를 설정한다. 이 파일을 작성하는 방법은 `.gitignore` 파일을 작성하는 방법과 같다. 그리고 `core.excludesfile`에 설정한 파일과 저장소 안에 있는 `.gitignore` 파일은 둘 다 사용된다.

You can put patterns in your project’s `.gitignore` file to have Git not see them as untracked files or try to stage them when you run `git add` on them, as discussed in Chapter 2. However, if you want another file outside of your project to hold those values or have extra values, you can tell Git where that file is with the `core.excludesfile` setting. Simply set it to the path of a file that has content similar to what a `.gitignore` file would have.

#### help.autocorrect ####

이 옵션은 Git 1.6.1 버전부터 사용할 수 있다. 명령어를 잘못 입력하면 Git 1.6에서는 메시지를 다음과 같이 보여 준다:

This option is available only in Git 1.6.1 and later. If you mistype a command in Git 1.6, it shows you something like this:

	$ git com
	git: 'com' is not a git-command. See 'git --help'.

	Did you mean this?
	     commit

그러나 `help.autocorrect`를 1로 설정한 상태에서 명령어를 잘못 입력하면 Git은 자동으로 그 명령어를 찾아서 실행해준다. 단, 비슷한 명령어가 딱 하나 있을 때에만 실행된다.

If you set `help.autocorrect` to 1, Git will automatically run the command if it has only one match under this scenario.

### 컬러 터미널 ###

사람이 쉽게 인식할 수 있도록 터미널에 결과를 컬러로 출력할 수 있다. 터미널 컬러와 관련된 옵션이 많아 꼼꼼하게 설정할 수 있다.

Git can color its output to your terminal, which can help you visually parse the output quickly and easily. A number of options can help you set the coloring to your preference.

#### color.ui ####

`color.ui`를 true로 설정하면 Git은 결과를 자동으로 색칠한다. 물론 무엇을 어떻게 색칠할지 꼼꼼하게 설정할 수 있지만, 이 옵션을 켜기만 해도 그냥 기본 컬러로 터미널이 칠해진다.

Git automatically colors most of its output if you ask it to. You can get very specific about what you want colored and how; but to turn on all the default terminal coloring, set `color.ui` to true:

	$ git config --global color.ui true

이 옵션을 켜면 Git은 터미널에 컬러로 결과를 출력한다. 이 값을 false로 설정하면 절대 컬러로 출력하지 않는다. 결과를 파일로 리다이렉트하거나 다른 프로그램으로 보낼(Piping. 파이프라인)때도 그렇다. 이 설정은 1.5.5 버전에 이르러 추가됐고 예전 버전을 사용하고 있다면 해당 요소마다 직접 컬러를 지정해주어야 한다.

When that value is set, Git colors its output if the output goes to a terminal. Other possible settings are false, which never colors the output, and always, which sets colors all the time, even if you’re redirecting Git commands to a file or piping them to another command. This setting was added in Git version 1.5.5; if you have an older version, you’ll have to specify all the color settings individually.

`color.ui = always`라고 설정하면 결과를 리다이렉트할 때에도 컬러 코드가 출력된다. 이렇게까지 설정해야 하는 경우는 매우 드물다. 대신 Git 명령에 `--color` 옵션을 주고 어떻게 출력할지 그때그때 정해줄 수 있다. 보통은 `color.ui = true` 만으로도 충분하다.

You’ll rarely want `color.ui = always`. In most scenarios, if you want color codes in your redirected output, you can instead pass a `--color` flag to the Git command to force it to use color codes. The `color.ui = true` setting is almost always what you’ll want to use.

#### `color.*` ####

좀 더 꼼꼼하게 컬러를 설정하거나 예전 버전이라서 `color.ui` 옵션을 사용할 수 없으면 다음과 같이 종류별로 설정할 수 있다. 모두 `true`, `false`, `always` 중 하나를 고를 수 있다:

If you want to be more specific about which commands are colored and how, or you have an older version, Git provides verb-specific coloring settings. Each of these can be set to `true`, `false`, or `always`:

	color.branch
	color.diff
	color.interactive
	color.status

또한 각 옵션의 컬러를 직접 지정할 수도 있다. 다음과 같이 설정하면 diff 명령에서 meta 정보의 foreground는 blue, background는 black, text는 bold로 바꿀 수 있다:

In addition, each of these has subsettings you can use to set specific colors for parts of the output, if you want to override each color. For example, to set the meta information in your diff output to blue foreground, black background, and bold text, you can run

	$ git config --global color.diff.meta “blue black bold”

컬러는 normal, black, red, green, yellow, blue, magenta, cyan, white 중에서 고를 수 있고 bold 같은 text 속성은 bold, dim, ul, blink, reverse 중에서 고를 수 있다.

You can set the color to any of the following values: normal, black, red, green, yellow, blue, magenta, cyan, or white. If you want an attribute like bold in the previous example, you can choose from bold, dim, ul, blink, and reverse.

`git config` Manpage를 보면 어떤 설정거리가 있는지 자세히 나온다.

See the `git config` manpage for all the subsettings you can configure, if you want to do that.

### 다른 Merge, Diff 도구 사용하기 ###

Git에 들어 있는 diff 말고 다른 도구로 바꿀 수 있다. GUI 기반의 화려한 것으로 바꿔서 좀 더 편리하게 충돌을 해결할 수 있다. 여기서는 Perforce의 Merge 도구인 P4Merge로 설정하는 것을 보여준다. P4Merge는 무료인데다 꽤 괜찮다.

Although Git has an internal implementation of diff, which is what you’ve been using, you can set up an external tool instead. You can also set up a graphical merge conflict-resolution tool instead of having to resolve conflicts manually. I’ll demonstrate setting up the Perforce Visual Merge Tool (P4Merge) to do your diffs and merge resolutions, because it’s a nice graphical tool and it’s free.

P4Merge는 중요 플랫폼을 모두 지원하기 때문에 대부분의 환경에서 사용할 수 있다. 여기서는 Mac과 Linux 시스템에 설치하는 것을 보여준다. Windows에서 사용하려면 `/usr/local/bin` 경로만 Windows에서의 경로로 바꿔주면 된다.

If you want to try this out, P4Merge works on all major platforms, so you should be able to do so. I’ll use path names in the examples that work on Mac and Linux systems; for Windows, you’ll have to change `/usr/local/bin` to an executable path in your environment.

다음 페이지에서 P4Merge를 내려 받는다.

You can download P4Merge here:

	http://www.perforce.com/perforce/downloads/component.html

먼저 P4Merge에 쓸 wrapper 스크립트를 만든다. 나는 Mac 사용자라서 Mac 경로를 사용한다. 어떤 시스템이든 `p4merge` 명령이 설치된 경로를 사용하면 된다. extMerge라는 Merge용 Wrapper 스크립트를 만들고 이 스크립트로 넘어오는 모든 인자를 p4merge 프로그램으로 넘긴다:

To begin, you’ll set up external wrapper scripts to run your commands. I’ll use the Mac path for the executable; in other systems, it will be where your `p4merge` binary is installed. Set up a merge wrapper script named extMerge that calls your binary with all the arguments provided:

	$ cat /usr/local/bin/extMerge
	#!/bin/sh
	/Applications/p4merge.app/Contents/MacOS/p4merge $*

그리고 diff용 Wrapper도 만든다. 이 스크립트로 넘어오는 인자는 총 7 개지만 그 중 2 개만 Merge Wrapper로 넘긴다. Git이 diff 프로그램에 넘겨주는 인자는 다음과 같다:

The diff wrapper checks to make sure seven arguments are provided and passes two of them to your merge script. By default, Git passes the following arguments to the diff program:

	path old-file old-hex old-mode new-file new-hex new-mode

이 중에서 `old-file`과 `new-file` 만 사용할 것이기 때문에 wrapper script를 만들어야 한다:

Because you only want the `old-file` and `new-file` arguments, you use the wrapper script to pass the ones you need.

	$ cat /usr/local/bin/extDiff 
	#!/bin/sh
	[ $# -eq 7 ] && /usr/local/bin/extMerge "$2" "$5"

이 두 스크립트에 실행 권한을 부여한다:

You also need to make sure these tools are executable:

	$ sudo chmod +x /usr/local/bin/extMerge 
	$ sudo chmod +x /usr/local/bin/extDiff

Git config 파일에 이 스크립트를 모두 추가해야 한다. 설정해야 하는 옵션이 좀 많다. `merge.tool`로 무슨 merge 도구를 사용할지, `mergetool.*.cmd`로 실제로 어떻게 명령어를 실행할지, `mergetool.trustExitCode`로 merge 도구가 반환하는 exit 코드가 merge의 성공여부를 나타내는지, `diff.external`은 diff할 때 실행할 명령어가 무엇인지를 설정할 때 사용한다. 모두 `git config` 명령으로 설정한다:

Now you can set up your config file to use your custom merge resolution and diff tools. This takes a number of custom settings: `merge.tool` to tell Git what strategy to use, `mergetool.*.cmd` to specify how to run the command, `mergetool.trustExitCode` to tell Git if the exit code of that program indicates a successful merge resolution or not, and `diff.external` to tell Git what command to run for diffs. So, you can either run four config commands

	$ git config --global merge.tool extMerge
	$ git config --global mergetool.extMerge.cmd \
	    'extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"'
	$ git config --global mergetool.trustExitCode false
	$ git config --global diff.external extDiff

`~/.gitconfig/` 파일을 직접 편집해도 된다:

or you can edit your `~/.gitconfig` file to add these lines:

	[merge]
	  tool = extMerge
	[mergetool "extMerge"]
	  cmd = extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"
	  trustExitCode = false
	[diff]
	  external = extDiff

설정을 완료하고 나서 다음과 같이 diff 명령어를 실행해본다: 

After all this is set, if you run diff commands such as this:
	
	$ git diff 32d1776b1^ 32d1776b1

diff 결과가 터미널에 출력되는 대신 P4Merge가 실행된다. 그리고 그림 7-1 처럼 그 프로그램 안에서 보여준다:

Instead of getting the diff output on the command line, Git fires up P4Merge, which looks something like Figure 7-1.

Insert 18333fig0701.png 
그림 7-1. P4Merge.

브랜치를 Merge할 때 충돌이 나면 `git mergetool` 명령을 실행한다. 이 명령을 실행하면 GUI 도구로 충돌을 해결할 수 있도록 P4Merge를 실행해준다.

If you try to merge two branches and subsequently have merge conflicts, you can run the command `git mergetool`; it starts P4Merge to let you resolve the conflicts through that GUI tool.

wrapper를 만들어 설정하면 다른 diff, merge 도구로 바꾸기 쉬워진다. 예를 들어, KDiff3를 사용하도록 extDiff와 extMerge 스크립트를 수정한다:

The nice thing about this wrapper setup is that you can change your diff and merge tools easily. For example, to change your extDiff and extMerge tools to run the KDiff3 tool instead, all you have to do is edit your extMerge file:

	$ cat /usr/local/bin/extMerge
	#!/bin/sh	
	/Applications/kdiff3.app/Contents/MacOS/kdiff3 $*

이제부터 Git은 diff 결과를 보여주거나 충돌을 해결할 때 KDiff3 도구를 사용한다.

Now, Git will use the KDiff3 tool for diff viewing and merge conflict resolution.

어떤 Merge 도구는 Git에 미리 cmd 설정이 들어 있다. 그래서 cmd 설정 없이 사용할 수 있는 것도 있다. kdiff3, opendiff, tkdiff, meld, xxdiff, emerge, vimdiff, gvimdiff는 cmd 설정 없이 merge 도구로 사용할 수 있다. diff 도구로는 다른 것을 사용하지만 merge 도구로는 KDiff3를 사용하고 싶은 경우에는 kdiff3 명령을 실행경로로 넣고 다음과 같이 설정하기만 하면 된다:

Git comes preset to use a number of other merge-resolution tools without your having to set up the cmd configuration. You can set your merge tool to kdiff3, opendiff, tkdiff, meld, xxdiff, emerge, vimdiff, or gvimdiff. If you’re not interested in using KDiff3 for diff but rather want to use it just for merge resolution, and the kdiff3 command is in your path, then you can run

	$ git config --global merge.tool kdiff3

extMerge와 extDiff 파일을 만들지 않으면 KDiff3를 Merge 도구로 사용하고 Git에 원래 들어 있던 것을 diff 도구로 사용한다.

If you run this instead of setting up the extMerge and extDiff files, Git will use KDiff3 for merge resolution and the normal Git diff tool for diffs.

### 소스 포맷과 공백 ###

협업할 때 겪는 소스 포맷(Formmatting)과 공백 문제는 미묘하고 난해하다. 동료 사이에 사용하는 플랫폼이 다를 때는 특히 더 심하다. 다른 사람이 보내온 Patch는 공백 문자가 미묘하게 다를 확률이 높다. 편집기가 몰래 공백문자를 추가해 버릴 수도 있고 크로스-플랫폼 프로젝트에서 윈도우 개발자가 줄 끝에 CR(Carriage-Return) 문자를 추가해 버렸을 수도 있다. Git에는 이 이슈를 돕는 몇 가지 설정이 있다.

Formatting and whitespace issues are some of the more frustrating and subtle problems that many developers encounter when collaborating, especially cross-platform. It’s very easy for patches or other collaborated work to introduce subtle whitespace changes because editors silently introduce them or Windows programmers add carriage returns at the end of lines they touch in cross-platform projects. Git has a few configuration options to help with these issues.

#### core.autocrlf ####

윈도우에서 개발하는 동료와 함께 일하면 줄바꿈(New Line) 문자 문제가 생긴다. 윈도우는 줄바꿈 문자로 CR(Carriage-Return)과 LF(LineFeed) 문자를 둘 다 사용하지만, Mac과 Linux는 LF 문자만 사용한다. 아무것도 아닌 것 같지만, 크로스 플랫폼 프로젝트에서는 꽤 성가신 문제다.

If you’re programming on Windows or using another system but working with people who are programming on Windows, you’ll probably run into line-ending issues at some point. This is because Windows uses both a carriage-return character and a linefeed character for newlines in its files, whereas Mac and Linux systems use only the linefeed character. This is a subtle but incredibly annoying fact of cross-platform work. 

Git은 커밋할 때 자동으로 CRLF를 LF로 변환해주고 반대로 Checkout할 때 LF를 CRLF로 변환해 주는 기능이 있다. `core.autocrlf` 설정으로 이 기능을 켤 수 있다. Windows에서 이 값을 true로 설정하면 Checkout할 때 LF 문자가 CRLR 문자로 변환된다:

Git can handle this by auto-converting CRLF line endings into LF when you commit, and vice versa when it checks out code onto your filesystem. You can turn on this functionality with the `core.autocrlf` setting. If you’re on a Windows machine, set it to `true` — this converts LF endings into CRLF when you check out code:

	$ git config --global core.autocrlf true

줄바꿈 문자로 LF를 사용하는 Linux와 Mac에서는 Checkout할 때 Git이 LF를 CRLF로 변환할 필요가 없다. 게다가 우연히 CRLF가 들어간 파일이 저장소에 들어 있어도 Git이 알아서 고쳐주면 좋을 것이다. `core.autocrlf` 값을 input으로 설정하면 커밋할 때만 CRLF를 LF로 변환한다:

If you’re on a Linux or Mac system that uses LF line endings, then you don’t want Git to automatically convert them when you check out files; however, if a file with CRLF endings accidentally gets introduced, then you may want Git to fix it. You can tell Git to convert CRLF to LF on commit but not the other way around by setting `core.autocrlf` to input:

	$ git config --global core.autocrlf input

이 설정을 이용하면 Windows에서는 CRLF를 사용하고 Mac, Linux, 저장소에서는 LF를 사용할 수 있다.

This setup should leave you with CRLF endings in Windows checkouts but LF endings on Mac and Linux systems and in the repository.

Windows 플랫폼에서만 개발한다면 이 기능을 끌 수 있다. 이 옵션을 `false`라고 설정하면 CR 문자도 저장소에도 저장된다:

If you’re a Windows programmer doing a Windows-only project, then you can turn off this functionality, recording the carriage returns in the repository by setting the config value to `false`:

	$ git config --global core.autocrlf false

#### core.whitespace ####

Git에는 공백 문자를 다루는 방법으로 네 가지가 미리 정의돼 있다. 두 가지는 기본적으로 켜져 있지만 끌 수 있고 나머지 두 가지는 꺼져 있지만 켤 수 있다.

Git comes preset to detect and fix some whitespace issues. It can look for four primary whitespace issues — two are enabled by default and can be turned off, and two aren’t enabled by default but can be activated.

먼저 기본적으로 켜져 있는 것을 살펴보자. `trailing-space`는 각 줄 끝에 공백이 있는지 찾고 `space-before-tab`은 모든 줄 처음에 tab보다 공백이 먼저 나오는지 찾는다.

The two that are turned on by default are `trailing-space`, which looks for spaces at the end of a line, and `space-before-tab`, which looks for spaces before tabs at the beginning of a line.

기본적으로 꺼져 있는 나머지 두 개는 `indent-with-non-tab`과 `cr-at-eol`이다. `intent-with-non-tab`은 tab이 아니라 공백 8자 이상으로 시작하는 줄이 있는지 찾고 `cr-at-eol`은 줄 끝에 CR 문자가 있어도 괜찮다고 Git에 알리는 것이다.

The two that are disabled by default but can be turned on are `indent-with-non-tab`, which looks for lines that begin with eight or more spaces instead of tabs, and `cr-at-eol`, which tells Git that carriage returns at the end of lines are OK.

`core.whitespace` 옵션으로 이 네 가지 방법을 켜고 끌 수 있다. 설정에서 해당 옵션을 빼버리거나 이름이 `-`로 시작하면 기능이 꺼진다. 예를 들어, 다른 건 다 켜고 `cr-at-eol` 옵션만 끄려면 다음과 같이 설정한다:

You can tell Git which of these you want enabled by setting `core.whitespace` to the values you want on or off, separated by commas. You can disable settings by either leaving them out of the setting string or prepending a `-` in front of the value. For example, if you want all but `cr-at-eol` to be set, you can do this:

	$ git config --global core.whitespace \
	    trailing-space,space-before-tab,indent-with-non-tab

`git diff` 명령을 실행할 때 Git은 이 설정에 따라 검사하고 쉽게 수정해서 커밋할 수 있도록 컬러로 표시해준다. `git apply` 명령으로 Patch를 적용할 때도 이 설정을 이용할 수 있다. Patch가 설정한 공백문자 정책에 들어맞는지 확인하려면 다음과 같이 명령어를 실행한다:

Git will detect these issues when you run a `git diff` command and try to color them so you can possibly fix them before you commit. It will also use these values to help you when you apply patches with `git apply`. When you’re applying patches, you can ask Git to warn you if it’s applying patches with the specified whitespace issues:

	$ git apply --whitespace=warn <patch>

아니면 Git이 자동으로 고치도록 할 수 있다:

Or you can have Git try to automatically fix the issue before applying the patch:

	$ git apply --whitespace=fix <patch>

이 옵션은 `git rebase` 명령에서도 사용할 수 있다. 공백 문제가 있는 커밋을 서버로 Push하기 전에  `--whitespace=fix` 옵션을 주고 Rebase하면 Git은 다시 Patch를 적용하면서 공백을 설정한대로 고친다.

These options apply to the git rebase option as well. If you’ve committed whitespace issues but haven’t yet pushed upstream, you can run a `rebase` with the `--whitespace=fix` option to have Git automatically fix whitespace issues as it’s rewriting the patches.

### 서버 설정 ###

서버 설정은 많지 않지만, 꼭 짚고 넘어가야 하는 것이 몇 개 있다. 

Not nearly as many configuration options are available for the server side of Git, but there are a few interesting ones you may want to take note of.

#### receive.fsckObjects ####

Git은 Push할 때 기본적으로 개체를 검증하지(check for consistency) 않는다. 하지만 Push할 때마다 각 개체가 SHA-1 체크섬에 맞는지, 가리키고 있는 개체가 괜찮은지 검사하게 할 수 있다. 개체를 점검하는 것은 상대적으로 느려서 Push하는 시간이 늘어 난다. 얼마나 늘어 나는지는 저장소 크기와 Push하는 양에 달렸다. `receive.fsckOBjects` 값을 true로 설정하면 Push할 때마다 Git이 검증한다.

By default, Git doesn’t check for consistency all the objects it receives during a push. Although Git can check to make sure each object still matches its SHA-1 checksum and points to valid objects, it doesn’t do that by default on every push. This is a relatively expensive operation and may add a lot of time to each push, depending on the size of the repository or the push. If you want Git to check object consistency on every push, you can force it to do so by setting `receive.fsckObjects` to true:

	$ git config --system receive.fsckObjects true

이렇게 설정하면 문제 있는 클라이언트가 잘못된 데이터를 Push하지 못하도록 Git은 매 Push마다 검증한다.

Now, Git will check the integrity of your repository before each push is accepted to make sure faulty clients aren’t introducing corrupt data.

#### receive.denyNonFastForwards ####

이미 Push한 커밋을 Rebase했을 때 다시 Push하지 못하게 할 수 있다. 또 원격 브랜치가 가리키는 커밋을 모르는 브랜치를 그 원격 브랜치로 Push하지 못하게도 할 수 있다. 보통은 이런 정책이 좋지만 `git push` 명령에 `-f` 옵션을 주면 강제로 Push할 수 있다.

If you rebase commits that you’ve already pushed and then try to push again, or otherwise try to push a commit to a remote branch that doesn’t contain the commit that the remote branch currently points to, you’ll be denied. This is generally good policy; but in the case of the rebase, you may determine that you know what you’re doing and can force-update the remote branch with a `-f` flag to your push command.

하지만 강제로 Push하지 못하게 할 수도 있다. `receive.denyNonFastForwards` 옵션으로 Fast-forward로 Push할 수 없는 브랜치는 아예 Push하지 못하게 할 수 있다:

To disable the ability to force-update remote branches to non-fast-forward references, set `receive.denyNonFastForwards`:

	$ git config --system receive.denyNonFastForwards true

사용자마다 다른 정책을 적용하고 싶다면 서버 훅을 사용 해야 한다. 서버의 receive 훅으로 할 수 있고 이 훅도 이 장에서 설명한다.

The other way you can do this is via server-side receive hooks, which I’ll cover in a bit. That approach lets you do more complex things like deny non-fast-forwards to a certain subset of users.

#### receive.denyDeletes ####

`receive.denyNonFastForwards`와 비슷한 정책으로 `receive.denyDeletes`라는 것이 있다. 이 설정을 켜면 브랜치를 삭제하는 Push가 거절된다. Git 1.6.1부터 receive.denyDeletes를 사용할 수 있다:

One of the workarounds to the `denyNonFastForwards` policy is for the user to delete the branch and then push it back up with the new reference. In newer versions of Git (beginning with version 1.6.1), you can set `receive.denyDeletes` to true:

	$ git config --system receive.denyDeletes true

이제 브랜치나 Tag를 삭제하는 Push는 거절된다. 아무도 삭제할 수 없다. 원격 브랜치를 삭제하려면 직접 손으로 server의 ref 파일을 삭제해야 한다. 그리고 ACL로 사용자마다 다른 정책을 적용 시킬 수도 방법도 있다. 이 방법은 이 장 끝부분에서 다룬다.

This denies branch and tag deletion over a push across the board — no user can do it. To remove remote branches, you must remove the ref files from the server manually. There are also more interesting ways to do this on a per-user basis via ACLs, as you’ll learn at the end of this chapter.

## Git Attribute ##

경로마다 다른 설정을 적용할 수 있기 때문에 디렉토리와 파일 단위로 다른 설정을 적용할 수 있다. 이렇게 경로별로 설정하는 것을 'Git Attribute'라고 부른다. 이 설정은 `.gitattributes`라는 파일에 저장하고 아무 디렉토리에나 둘 수 있지만 보통은 프로젝트 최상위 디렉토리에 둔다. 그리고 이 파일을 커밋하고 싶지 않으면 파일을 `.gitattributes`가 아니라 `.git/info/attributes`로 만든다.

Some of these settings can also be specified for a path, so that Git applies those settings only for a subdirectory or subset of files. These path-specific settings are called Git attributes and are set either in a `.gitattributes` file in one of your directories (normally the root of your project) or in the `.git/info/attributes` file if you don’t want the attributes file committed with your project.

이 Attribute로 Merge는 어떻게 할지, 텍스트가 아닌 파일은 어떻게 Diff할지, checkin/checkout할 때 어떻게 필터링할지 정해줄 수 있다. 이 절에서는 설정할 수 있는 Attribute가 어떤 것이 있는지, 그리고 어떻게 설정하는지 배우고 예제를 살펴본다.

Using attributes, you can do things like specify separate merge strategies for individual files or directories in your project, tell Git how to diff non-text files, or have Git filter content before you check it into or out of Git. In this section, you’ll learn about some of the attributes you can set on your paths in your Git project and see a few examples of using this feature in practice.

### 바이너리 파일 ###

어떤 파일이 바이너리 파일인지 Attribute로 Git에게 알려줄 수 있는데 이 Attribute는 좀 좋다. 기본적으로 Git은 바이너리 파일이 어떤 파일인지 알지 못한다. 그렇지만 Git이 파일을 어떻게 다뤄야 하는지 알려주는 명령어가 있다. 예를 들어 어떤 텍스트 파일은 기계가 만든 파일이라 diff할 수 없지만 어떤 바이너리 파일은 취급 방법을 Git에 알려 주면 diff할 수 있다.

One cool trick for which you can use Git attributes is telling Git which files are binary (in cases it otherwise may not be able to figure out) and giving Git special instructions about how to handle those files. For instance, some text files may be machine generated and not diffable, whereas some binary files can be diffed — you’ll see how to tell Git which is which.

#### 바이너리 파일이라고 알려주기 ####

사실 텍스트 파일이지만 만든 목적과 의도로 보면 바이너리 파일인 것이 있다. 예를 들어, Mac의 Xcode 프로젝트는 `.pbxproj`로 끝나는 파일을 만든다. 이 파일은 JSON 포맷이지만, IDE에서 설정 등을 디스크에 저장하는 파일이다. 본질적으로 모든 것이 ASCII인 텍스트 파일이지만 실제로는 간단한 데이터베이스이기 때문에 텍스트 파일처럼 취급할 수 없다. 그래서 여러 명이 이 파일을 동시에 수정하고 merge하면 diff는 도움이 안 된다. 이 파일은 기계가 읽고 쓰는 파일이기 때문에 바이너리 파일처럼 취급하는 것이 옳다.

Some files look like text files but for all intents and purposes are to be treated as binary data. For instance, Xcode projects on the Mac contain a file that ends in `.pbxproj`, which is basically a JSON (plain text javascript data format) dataset written out to disk by the IDE that records your build settings and so on. Although it’s technically a text file, because it’s all ASCII, you don’t want to treat it as such because it’s really a lightweight database — you can’t merge the contents if two people changed it, and diffs generally aren’t helpful. The file is meant to be consumed by a machine. In essence, you want to treat it like a binary file.

모든 `pbxproj` 파일을 바이너리로 파일로 취급하는 설정은 다음과 같다. `.gitattributes` 파일에 넣으면 된다:

To tell Git to treat all `pbxproj` files as binary data, add the following line to your `.gitattributes` file:

	*.pbxproj -crlf -diff

이제 Git은 CRLF 문제로 `pbxproj` 파일을 변환하지 않는다. `git show`나 `git diff` 같은 명령을 실행해도 통계를 계산하지도 않고 diff를 출력하지도 않는다. Git 1.6 부터는 `-crlf -diff`를 한 마디로 줄여서 표현할 수 있다:

Now, Git won’t try to convert or fix CRLF issues; nor will it try to compute or print a diff for changes in this file when you run git show or git diff on your project. In the 1.6 series of Git, you can also use a macro that is provided that means `-crlf -diff`:

	*.pbxproj binary

#### 바이너리 파일 Diff하기 ####

Git 1.6부터 바이너리 파일도 diff할 수 있게 됐다. 이 Attribute는 Git이 바이너리 파일을 텍스트 포맷으로 변환하고 그 결과를 diff로 비교하도록 하는 것이다.

In the 1.6 series of Git, you can use the Git attributes functionality to effectively diff binary files. You do this by telling Git how to convert your binary data to a text format that can be compared via the normal diff.

이 Attribute는 잘 알려지진 않았지만 끝내준다. 이 Attribute가 유용한 예제를 하나 살펴보자. 먼저 이 기술을 인류에게 알려진 가장 귀찮은 문제 중 하나인 Word 문서를 버전 관리하는 상황을 살펴보자. 모든 사람이 Word가 가장 끔찍한 편집기라고 말하지만 애석하게도 모두 Word를 사용한다. Git 저장소에 넣고 이따금 커밋하는 것만으로도 Word 문서의 버전을 관리할 수 있다. 그렇지만 `git diff`를 실행하면 다음과 같은 메시지를 볼 수 있을 뿐이다:

Because this is a pretty cool and not widely known feature, I’ll go over a few examples. First, you’ll use this technique to solve one of the most annoying problems known to humanity: version-controlling Word documents. Everyone knows that Word is the most horrific editor around; but, oddly, everyone uses it. If you want to version-control Word documents, you can stick them in a Git repository and commit every once in a while; but what good does that do? If you run `git diff` normally, you only see something like this:

	$ git diff 
	diff --git a/chapter1.doc b/chapter1.doc
	index 88839c4..4afcb7c 100644
	Binary files a/chapter1.doc and b/chapter1.doc differ

직접 파일을 하나하나 까보지 않으면 두 버전이 뭐가 다른지 알 수 없다. Git Attribute를 사용하면 이를 더 좋게 개선할 수 있다. `.gitattributes` 파일에 다음과 같은 내용을 추가한다:

You can’t directly compare two versions unless you check them out and scan them manually, right? It turns out you can do this fairly well using Git attributes. Put the following line in your `.gitattributes` file:

	*.doc diff=word

이것은 `*.doc` 파일의 두 버전이 무엇이 다른지 diff할 때 "word" 필터를 사용하라고 설정하는 것이다. 그럼 "word" 필터는 뭘까? 이 "word" 필터도 정의해야 한다. Word 문서에서 사람이 읽을 수 있는 텍스트를 추출해주는 `strings` 프로그램을 "word" 필터로 사용한다. 그러면 Word 문서를 diff할 수 있다:

This tells Git that any file that matches this pattern (.doc) should use the "word" filter when you try to view a diff that contains changes. What is the "word" filter? You have to set it up. Here you’ll configure Git to use the `strings` program to convert Word documents into readable text files, which it will then diff properly:

	$ git config diff.word.textconv strings

이제 Git은 확장자가 `.doc`인 파일의 스냅샷을 diff할 때 "word" 필터로 정의한 `strings` 프로그램을 사용한다. 이 프로그램은 Word파일을 텍스트 파일로 변환해 주기 때문에 diff할 수 있다.

Now Git knows that if it tries to do a diff between two snapshots, and any of the files end in `.doc`, it should run those files through the "word" filter, which is defined as the `strings` program. This effectively makes nice text-based versions of your Word files before attempting to diff them.

이 책의 1 장을 Word 파일로 만들어서 Git에 넣고 나서 단락 하나를 수정하고 저장하는 예를 살펴보자. `git diff`를 실행하면 어디가 달려졌는지 확인할 수 있다:

Here’s an example. I put Chapter 1 of this book into Git, added some text to a paragraph, and saved the document. Then, I ran `git diff` to see what changed:

	$ git diff
	diff --git a/chapter1.doc b/chapter1.doc
	index c1c8a0a..b93c9e4 100644
	--- a/chapter1.doc
	+++ b/chapter1.doc
	@@ -8,7 +8,8 @@ re going to cover Version Control Systems (VCS) and Git basics
	 re going to cover how to get it and set it up for the first time if you don
	 t already have it on your system.
	 In Chapter Two we will go over basic Git usage - how to use Git for the 80% 
	-s going on, modify stuff and contribute changes. If the book spontaneously 
	+s going on, modify stuff and contribute changes. If the book spontaneously 
	+Let's see if this works.

Git은 "Let's see if this works"가 추가됐다는 것을 정확하게 찾아 준다. 이것은 완벽하지는 않지만(마지막에 아무거나 왕창 집어넣지만 않으면) 어쨌든 잘 동작한다. 이 방법은 Word 문서를 텍스트로 더 잘 변환하는 프로그램이 있으면 좀 더 완벽해질 수 있다. Mac이나 Linux 같은 시스템에는 `strings`가 이미 설치되어 있기 때문에 당장 사용할 수 있다.

Git successfully and succinctly tells me that I added the string "Let’s see if this works", which is correct. It’s not perfect — it adds a bunch of random stuff at the end — but it certainly works. If you can find or write a Word-to-plain-text converter that works well enough, that solution will likely be incredibly effective. However, `strings` is available on most Mac and Linux systems, so it may be a good first try to do this with many binary formats.

이 방법으로 이미지 파일도 diff할 수 있다. 필터로 EXIF 정보를 추출해서 JPEG 파일을 비교한다. EXIF 정보는 대부분의 이미지 파일에 들어 있는 메타데이터다. `exiftool`이라는 프로그램을 설치하고 이미지 파일에서 메타데이터 텍스트를 추출한다. 그리고 그 결과를 diff해서 무엇이 달라졌는지 본다:

Another interesting problem you can solve this way involves diffing image files. One way to do this is to run JPEG files through a filter that extracts their EXIF information — metadata that is recorded with most image formats. If you download and install the `exiftool` program, you can use it to convert your images into text about the metadata, so at least the diff will show you a textual representation of any changes that happened:

	$ echo '*.png diff=exif' >> .gitattributes
	$ git config diff.exif.textconv exiftool

프로젝트에 들어 있는 이미지 파일을 새로 바꾸고 `git diff`를 실행하면 다음과 같이 보여준다:

If you replace an image in your project and run `git diff`, you see something like this:

	diff --git a/image.png b/image.png
	index 88839c4..4afcb7c 100644
	--- a/image.png
	+++ b/image.png
	@@ -1,12 +1,12 @@
	 ExifTool Version Number         : 7.74
	-File Size                       : 70 kB
	-File Modification Date/Time     : 2009:04:21 07:02:45-07:00
	+File Size                       : 94 kB
	+File Modification Date/Time     : 2009:04:21 07:02:43-07:00
	 File Type                       : PNG
	 MIME Type                       : image/png
	-Image Width                     : 1058
	-Image Height                    : 889
	+Image Width                     : 1056
	+Image Height                    : 827
	 Bit Depth                       : 8
	 Color Type                      : RGB with Alpha

이미지 파일의 크기와 해상도가 달라진 것을 쉽게 알 수 있다:

You can easily see that the file size and image dimensions have both changed.

### 키워드 치환 ###

SVN이나 CVS에 익숙한 사람들은 해당 시스템에서 사용하던 키워드 치환(Keyword Expansion) 기능을 찾는다. Git에서는 이것이 쉽지 않다. Git은 먼저 체크섬을 계산하고 커밋하기 때문에 그 커밋에 대한 정보를 가지고 파일을 수정할 수 없다. 하지만 Checkout할 때 그 정보가 자동으로 파일에 삽입되도록 했다가 다시 커밋할 때 삭제되도록 할 수 있다.

SVN- or CVS-style keyword expansion is often requested by developers used to those systems. The main problem with this in Git is that you can’t modify a file with information about the commit after you’ve committed, because Git checksums the file first. However, you can inject text into a file when it’s checked out and remove it again before it’s added to a commit. Git attributes offers you two ways to do this.

파일안에 `$Id$` 필드를 넣어주면 Blob의 SHA-1 체크섬을 자동으로 삽입시킬 수 있다. 이 필드를 파일에 넣으면 Git은 다음 번에 Checkout할 때부터 해당 Blob의 SHA-1 값으로 교체한다. 여기서 꼭 기억해야 할 것이 있는데, 교체되는 체크섬은 커밋의 것이 아니라 Blob 그 자체의 SHA 체크섬이다:

First, you can inject the SHA-1 checksum of a blob into an `$Id$` field in the file automatically. If you set this attribute on a file or set of files, then the next time you check out that branch, Git will replace that field with the SHA-1 of the blob. It’s important to notice that it isn’t the SHA of the commit, but of the blob itself:

	$ echo '*.txt ident' >> .gitattributes
	$ echo '$Id$' > test.txt

이 파일을 다음에 Checkout할 때 Git은 SHA 값을 삽입해준다:

The next time you check out this file, Git injects the SHA of the blob:

	$ rm text.txt
	$ git checkout -- text.txt
	$ cat test.txt 
	$Id: 42812b7653c7b88933f8a9d6cad0ca16714b9bb3 $

하지만 이것은 별로 유용하지 않다. CVS나 SVN의 키워드 치환(Keyword Substitution)을 써봤으면 날짜(Datestamp)도 가능하다는 것을 알고 있을 것이다. SHA는 그냥 해시일 뿐이라 식별할 수 있을 뿐이지 다른 것을 알려주진 않는다. SHA만으로 예전 것보다 새 것인지 오래된 것인지는 알 수 없다.

However, that result is of limited use. If you’ve used keyword substitution in CVS or Subversion, you can include a datestamp — the SHA isn’t all that helpful, because it’s fairly random and you can’t tell if one SHA is older or newer than another.

Commit/Checkout할 때 사용할 필터를 직접 만들어 쓸 수 있는데, 방향에 따라 "clean" 필터와 "smudge" 필터라고 부른다. ".gitattributes" 파일에 파일 경로마다 다른 필터를 설정할 수 있다. Checkout할 때 파일을 처리하는 것이 "smudge" 필터이고(그림 7-2) 커밋할 때 처리하는 필터가 "clean" 필터이다. 이 필터로 할 수 있는 일은 무궁무진하다.

It turns out that you can write your own filters for doing substitutions in files on commit/checkout. These are the "clean" and "smudge" filters. In the `.gitattributes` file, you can set a filter for particular paths and then set up scripts that will process files just before they’re checked out ("smudge", see Figure 7-2) and just before they’re committed ("clean", see Figure 7-3). These filters can be set to do all sorts of fun things.

Insert 18333fig0702.png 
Figure 7-2. The “smudge” filter is run on checkout. / "smudge" 필터는 Checkout할 때 실행된다.

Insert 18333fig0703.png 
Figure 7-3. The “clean” filter is run when files are staged. / "clean" 필터는 파일을 Stage할 때 실행된다.

커밋하기 전에 `indent` 프로그램으로 C 코드 전부를 필터링하지만 커밋 메시지는 단순한 예제를 보자. `*.c` 파일은 indent 필터를 사용하도록 `.gitattributes` 파일에 설정한다:

The original commit message for this functionality gives a simple example of running all your C source code through the `indent` program before committing. You can set it up by setting the filter attribute in your `.gitattributes` file to filter `*.c` files with the "indent" filter:

	*.c     filter=indent

다음은 "indent" 필터에 사용하는 smudge와 clean이 각각 무엇인지 설정한다:

Then, tell Git what the "indent"" filter does on smudge and clean:

	$ git config --global filter.indent.clean indent
	$ git config --global filter.indent.smudge cat

`*.c` 파일을 커밋하면 `indent` 프로그램을 통해서 커밋되고 다시 Checkout하기 전에는 `cat` 프로그램을 통해 Checkout된다. `cat`은 입력된 데이터를 그대로 다시 내보내는, 사실 아무것도 안하는 프로그램이다. 이 설정으로 모든 C 소스 파일은 `indent` 프로그램을 통해 커밋된다.

In this case, when you commit files that match `*.c`, Git will run them through the indent program before it commits them and then run them through the `cat` program before it checks them back out onto disk. The `cat` program is basically a no-op: it spits out the same data that it gets in. This combination effectively filters all C source code files through `indent` before committing.

이제 RCS 처럼 `$Date$`를 치환하는 것을 살펴보자. 이를 하려면 간단한 스크립트가 하나 필요하다. 이 스크립트는 표준 입력을 읽어서 $Date$ 필드를 해당 프로젝트의 마지막 커밋 일자를 구한 날자로 치환한다. 다음은 Ruby로 구현한 스크립트다:

Another interesting example gets `$Date$` keyword expansion, RCS style. To do this properly, you need a small script that takes a filename, figures out the last commit date for this project, and inserts the date into the file. Here is a small Ruby script that does that:

	#! /usr/bin/env ruby
	data = STDIN.read
	last_date = `git log --pretty=format:"%ad" -1`
	puts data.gsub('$Date$', '$Date: ' + last_date.to_s + '$')

`git log` 명령으로 마지막 커밋정보를 얻어서 STDIN에서 `$Date$` 스트링을 찾아서 치환한다. 사용하기 편한 언어로 스크립트를 만들면 된다. 이 스크립트의 이름을 `expand_date`라고 짓고 실행 경로에 넣는다. 그리고 `dater`라는 Git 필터를 정의한다. Checkout시 실행하는 smudge 필터로 `expand_date`를 사용하고 커밋할 때 실행하는 clean 필터는 Perl을 사용한다:

All the script does is get the latest commit date from the `git log` command, stick that into any `$Date$` strings it sees in stdin, and print the results — it should be simple to do in whatever language you’re most comfortable in. You can name this file `expand_date` and put it in your path. Now, you need to set up a filter in Git (call it `dater`) and tell it to use your `expand_date` filter to smudge the files on checkout. You’ll use a Perl expression to clean that up on commit:

	$ git config filter.dater.smudge expand_date
	$ git config filter.dater.clean 'perl -pe "s/\\\$Date[^\\\$]*\\\$/\\\$Date\\\$/"'

이 Perl 코드는 `$Date$` 스트링에 있는 문자를 제거해서 원래대로 복원한다. 이제 필터가 준비 됐으니 `$Date$` 키워드가 들어 있는 파일을 만들고 Git Attribute를 설정해서 새 필터를 시험한다:

This Perl snippet strips out anything it sees in a `$Date$` string, to get back to where you started. Now that your filter is ready, you can test it by setting up a file with your `$Date$` keyword and then setting up a Git attribute for that file that engages the new filter:

	$ echo '# $Date$' > date_test.txt
	$ echo 'date*.txt filter=dater' >> .gitattributes

이를 커밋하고 파일을 다시 Checkout 하면 해당 키워드가 적절히 치환된 것을 볼 수 있다:

If you commit those changes and check out the file again, you see the keyword properly substituted:

	$ git add date_test.txt .gitattributes
	$ git commit -m "Testing date expansion in Git"
	$ rm date_test.txt
	$ git checkout date_test.txt
	$ cat date_test.txt
	# $Date: Tue Apr 21 07:26:52 2009 -0700$

이것은 매우 강력해서 두루두루 넓게 적용할 수 있다. `.gitattributes` 파일은 커밋할 것이기 때문에 드라이버(여기서는 `dater`)가 없는 사람에게도 배포된다. `dater`가 없으면 에러가 나기 때문에 필터를 만들때 이 같은 예외 상황도 고려해서 항상 잘 동작하게 해야 한다.

You can see how powerful this technique can be for customized applications. You have to be careful, though, because the `.gitattributes` file is committed and passed around with the project but the driver (in this case, `dater`) isn’t; so, it won’t work everywhere. When you design these filters, they should be able to fail gracefully and have the project still work properly.

### 저장소 익스포트하기 ###

프로젝트를 익스포트해서 아카이브를 만들 때에도 Git Attribute가 유용하다.

Git attribute data also allows you to do some interesting things when exporting an archive of your project.

#### export-ignore ####

아카이브를 만들때 제외시킬 파일이나 디렉토리가 무엇인지 설정할 수 있다. 특정 디렉토리나 파일을 프로젝트에는 포함시키고 아카이브에는 포함시키고 싶지 않을 때 `export-ignore` Attribute를 사용한다.

You can tell Git not to export certain files or directories when generating an archive. If there is a subdirectory or file that you don’t want to include in your archive file but that you do want checked into your project, you can determine those files via the `export-ignore` attribute.

예를 들어 `test/` 디렉토리에 테스트 파일들이 있다고 하자. 보통 tar 파일로 묶어서 익스포트할 때 테스트 파일은 포함시키지 않는다. Git Attribute 파일에 다음 라인을 추가하면 테스트 파일은 무시된다:

For example, say you have some test files in a `test/` subdirectory, and it doesn’t make sense to include them in the tarball export of your project. You can add the following line to your Git attributes file:

	test/ export-ignore

`git archive` 명령으로 tar 파일을 만들면 test 디렉토리는 아카이브에 포함되지 않는다.

Now, when you run git archive to create a tarball of your project, that directory won’t be included in the archive.

#### export-subst ####

아카이브를 만들 때에도 키워드를 치환할 수 있다. 파일을 하나 만들고 거기에 `$Format:$` 스트링을 넣으면 Git이 치환해준다. 이 스트링에 `--pretty=format` 옵션에 사용하는 것과 같은 포맷 코드를 넣을 수 있다. `--pretty=format`은 2 장에서 배웠다. 예를 들어 `LAST_COMMIT`이라는 파일을 만들고 `git archive` 명령을 실행할 때 자동으로 이 파일에 마지막 커밋 날짜가 삽입되게 하려면 다음과 같이 해야 한다:

Another thing you can do for your archives is some simple keyword substitution. Git lets you put the string `$Format:$` in any file with any of the `--pretty=format` formatting shortcodes, many of which you saw in Chapter 2. For instance, if you want to include a file named `LAST_COMMIT` in your project, and the last commit date was automatically injected into it when `git archive` ran, you can set up the file like this:

	$ echo 'Last commit date: $Format:%cd$' > LAST_COMMIT
	$ echo "LAST_COMMIT export-subst" >> .gitattributes
	$ git add LAST_COMMIT .gitattributes
	$ git commit -am 'adding LAST_COMMIT file for archives'

`git archive` 명령으로 아카이브를 만들고 나서 이 파일을 열어보면 다음과 같이 보일 것이다:

When you run `git archive`, the contents of that file when people open the archive file will look like this:

	$ cat LAST_COMMIT
	Last commit date: $Format:Tue Apr 21 08:38:48 2009 -0700$

### Merge 전략 ###

파일마다 다른 Merge 전략을 사용하도록 설정할 수 있다. Merge할 때 충돌이 날 것 같은 파일이 있다고 하자. Git Attrbute로 이 파일만 항상 타인의 코드 말고 내 코드를 사용하도록 설정할 수 있다.

You can also use Git attributes to tell Git to use different merge strategies for specific files in your project. One very useful option is to tell Git to not try to merge specific files when they have conflicts, but rather to use your side of the merge over someone else’s.

Merge하는 브랜치가 다른 환경에서 운영하기 위해 만든 브랜치일때 유용하다. 이때는 환경 설정과 관련된 파일은 merge하지 않고 무시하는 게 편리하다. 두 브랜치에 database.xml이라는 데이터베이스 설정파일이 있는데 이 파일은 브랜치마다 다르다. datebase 설정 파일은 Merge하면 안 되기 때문에 Attribute를 다음과 같이 설정하면 이 파일은 그냥 두고 Merge한다.

This is helpful if a branch in your project has diverged or is specialized, but you want to be able to merge changes back in from it, and you want to ignore certain files. Say you have a database settings file called database.xml that is different in two branches, and you want to merge in your other branch without messing up the database file. You can set up an attribute like this:

	database.xml merge=ours

이제 Merge해도 database.xml 파일은 충돌하지 않는다:

If you merge in the other branch, instead of having merge conflicts with the database.xml file, you see something like this:

	$ git merge topic
	Auto-merging database.xml
	Merge made by recursive.

Merge했지만 database.xml은 원래 가지고 있던 파일 그대로다.

In this case, database.xml stays at whatever version you originally had.

## Git 훅 ##

Git도 다른 버전 관리 시스템처럼 어떤 이벤트가 생겼을 때 자동으로 특정 스크립트를 실행도록 할 수 있다. 이 훅은 클라이언트와 서버 두 가지로 나눌 수 있다. 클라이언트 훅은 커밋이나 Merge할 때 실행되고 서버 훅은 Push할 때 실행된다. 이 절에서는 어떤 훅이 있고 어떻게 사용하는지 배운다.

Like many other Version Control Systems, Git has a way to fire off custom scripts when certain important actions occur. There are two groups of these hooks: client side and server side. The client-side hooks are for client operations such as committing and merging. The server-side hooks are for Git server operations such as receiving pushed commits. You can use these hooks for all sorts of reasons, and you’ll learn about a few of them here.

### 훅 설치하기 ###

훅은 Git 디렉토리 밑에 `hooks`라는 디렉토리에 저장한다. 대부분 `.git/hooks`이라는 디렉토리다. 이 디렉토리에 가보면 Git이 자동으로 넣어준 매우 유용한 스크립트 예제가 몇 개 있다. 그리고 스크립트가 입력받는 값이 어떤 값인지 파일 안에 자세히 설명돼 있다. 모든 예제는 쉘과 Perl 스크립트로 작성돼 있지만 실행할 수만 있으면 되고 Ruby나 Python같은 다른 스크립트 언어로 만들어도 된다. Git 1.6 부터 예제 스크립트의 파일 이름이 `.sample`이라는 확장자가 붙어있다. 그래서 이름만 바꿔주면 그 훅을 사용할 수 있다. 1.6 이전 버전에서는 파일 이름과 상관없이 실행되지 않는 파일이 예제 파일이었다.

The hooks are all stored in the `hooks` subdirectory of the Git directory. In most projects, that’s `.git/hooks`. By default, Git populates this directory with a bunch of example scripts, many of which are useful by themselves; but they also document the input values of each script. All the examples are written as shell scripts, with some Perl thrown in, but any properly named executable scripts will work fine — you can write them in Ruby or Python or what have you. For post-1.6 versions of Git, these example hook files end with .sample; you’ll need to rename them. For pre-1.6 versions of Git, the example files are named properly but are not executable.

실행할 수 있는 스크립트 파일을 저장소의 `hooks` 디렉토리에 넣으면 훅 스크립트가 켜진다. 앞으로 계속 이 스크립트가 호출된다. 여기서 중요한 훅은 모두 설명할 것이다.

To enable a hook script, put a file in the `hooks` subdirectory of your Git directory that is named appropriately and is executable. From that point forward, it should be called. I’ll cover most of the major hook filenames here.

### 클라이언트 훅 ###

클라이언트 훅은 매우 다양하다. 이 절에서는 클라이언트 훅을 커밋 Workflow 훅, E-mail Workflow 훅, 그리고 나머지로 분류해서 설명한다.

There are a lot of client-side hooks. This section splits them into committing-workflow hooks, e-mail-workflow scripts, and the rest of the client-side scripts.

#### 커밋 Workflow 훅 #### 

먼저 커밋과 관련된 훅을 살펴보자. 커밋과 관련된 훅은 모두 네 가지다. `pre-commit` 훅은 커밋할 때 가장 먼저 호출되는 훅으로 커밋 메시지를 작성하기 전에 호출된다. 이 훅에서 커밋하는 Snapshot을 점검한다. 빠트린 것은 없는지, 테스트는 확실히 했는지 등을 검사한다. 커밋할 때 꼭 확인해야 할 게 있으면 이 훅으로 확인한다. 그리고 이 훅의 Exit 코드가 0이 아니면 커밋은 취소된다. 물론 `git commit --no-verify`라고 실행하면 이 훅을 일시적으로 생략할 수 있다. lint 같은 프로그램으로 코드 스타일을 검사하거나, 줄 끝의 공백 문자를 검사하거나(예제로 들어 있는 `pre-commit` 훅이 하는 게 이 일이다), 코드에 주석을 달았는지 검사하는 일은 이 훅으로 하는 것이 좋다.

The first four hooks have to do with the committing process. The `pre-commit` hook is run first, before you even type in a commit message. It’s used to inspect the snapshot that’s about to be committed, to see if you’ve forgotten something, to make sure tests run, or to examine whatever you need to inspect in the code. Exiting non-zero from this hook aborts the commit, although you can bypass it with `git commit --no-verify`. You can do things like check for code style (run lint or something equivalent), check for trailing whitespace (the default hook does exactly that), or check for appropriate documentation on new methods.

`prepare-commit-msg` 훅은 Git이 커밋 메시지를 생성하고 나서 편집기를 실행하기 전에 실행된다. 이 훅은 사람이 커밋 메시지를 수정하기 전에 먼저 프로그램으로 손보고 싶을 때 사용한다. 이 훅은 커밋 메시지가 들어 있는 파일의 경로, 커밋의 종류를 인자로 받는다. 그리고 최근 커밋을 수정할 때에는(Amending commit) SHA-1 값을 추가 인자로 더 받는다. 사실 이 훅은 일반 커밋에는 별로 필요 없고 커밋 메시지를 자동으로 생성하는 커밋에 좋다. 커밋 메시지에 템플릿을 적용하거나, Merge 커밋, Squash 커밋, Amend 커밋일 때 유용하다. 이 스크립트로 커밋 메시지 템플릿에 정보를 삽입할 수 있다. 

The `prepare-commit-msg` hook is run before the commit message editor is fired up but after the default message is created. It lets you edit the default message before the commit author sees it. This hook takes a few options: the path to the file that holds the commit message so far, the type of commit, and the commit SHA-1 if this is an amended commit. This hook generally isn’t useful for normal commits; rather, it’s good for commits where the default message is auto-generated, such as templated commit messages, merge commits, squashed commits, and amended commits. You may use it in conjunction with a commit template to programmatically insert information.

`commit-msg` 훅은 커밋 메시지가 들어있는 임시 파일의 경로를 인자로 받는다. 그리고 이 스크립트가 0이 아닌값 을 반환하면 Git은 커밋하지 않는다. 최종적으로 커밋되기 전에 이 훅에서 프로젝트 상태나 커밋 메시지를 검증한다. 이 장의 마지막 절에서 이 훅을 사용하는 예제를 보여줄 것이다. 커밋 메시지가 정책에 맞는지 검사하는 스크립트를 만들어 보자.

The `commit-msg` hook takes one parameter, which again is the path to a temporary file that contains the current commit message. If this script exits non-zero, Git aborts the commit process, so you can use it to validate your project state or commit message before allowing a commit to go through. In the last section of this chapter, I’ll demonstrate using this hook to check that your commit message is conformant to a required pattern.

커밋이 완료되면 `post-commit` 훅이 실행된다. 이 훅은 넘겨받는 인자가 하나도 없지만 `git log -1 HEAD` 명령으로 쉽게 정보를 얻을 수 있다. 일반적으로 이 스크립트는 커밋된 것을 누군가에게 알릴 때 사용한다.

After the entire commit process is completed, the `post-commit` hook runs. It doesn’t take any parameters, but you can easily get the last commit by running `git log -1 HEAD`. Generally, this script is used for notification or something similar.

이 Committing-workflow 스크립트는 어떤 Workflow에나 사용할 수 있고 정책을 강제할 때 유용하다. 클라이언트 훅은 개발자가 클라이언트에서 사용하는 것이다. 그래서 각 개발자에게 유용하지만 Clone할 때 복사되지 않기 때문에 직접 설치하고 관리해야 주어야 한다. 물론 정책을 서버 훅으로 만들고 정책을 잘 지키는지 Push할 때 검사해도 된다.

The committing-workflow client-side scripts can be used in just about any workflow. They’re often used to enforce certain policies, although it’s important to note that these scripts aren’t transferred during a clone. You can enforce policy on the server side to reject pushes of commits that don’t conform to some policy, but it’s entirely up to the developer to use these scripts on the client side. So, these are scripts to help developers, and they must be set up and maintained by them, although they can be overridden or modified by them at any time.

#### E-mail Workflow 훅 ####

E-mail Workflow에 해당하는 클라이언트 훅은 세 가지이다. 이 훅은 모두 `git am` 명령으로 실행되기 때문에 이 명령어를 사용할 일이 없으면 이 절은 읽지 않아도 된다. 하지만 언젠가는 `git format-patch` 명령으로 만든 Patch를 E-mail로 받는 날이 올지도 모른다.

You can set up three client-side hooks for an e-mail-based workflow. They’re all invoked by the `git am` command, so if you aren’t using that command in your workflow, you can safely skip to the next section. If you’re taking patches over e-mail prepared by `git format-patch`, then some of these may be helpful to you.

제일 먼저 실행하는 훅은 `applypatch-msg`이다. 이 훅의 인자는 Author가 보내온 커밋 메시지 파일 이름이다. 이 스크립트가 종료할 때 0이 아닌 값을 반환하면 Git은 Patch하지 않는다. 커밋 메시지가 규칙에 맞는지 확인하거나 자동으로 메시지를 수정할 때 이 훅을 사용한다.

The first hook that is run is `applypatch-msg`. It takes a single argument: the name of the temporary file that contains the proposed commit message. Git aborts the patch if this script exits non-zero. You can use this to make sure a commit message is properly formatted or to normalize the message by having the script edit it in place.

`git am` 으로 Patch할 때 두 번째로 실행되는 훅이 `pre-applypatch`이다. 이 훅은 인자가 없고 단순히 Patch를 적용한 후에 실행된다. 그래서 커밋할 스냅샷을 검사하는 데 사용한다. 이 스크립트로 테스트를 수행하고 파일을 검사할 수 있다. 뭔가 테스트에 실패하거나 뭔가 부족하면 0이 아닌 값을 반환시켜서 `git am` 명령을 취소시킬 수 있다.

The next hook to run when applying patches via `git am` is `pre-applypatch`. It takes no arguments and is run after the patch is applied, so you can use it to inspect the snapshot before making the commit. You can run tests or otherwise inspect the working tree with this script. If something is missing or the tests don’t pass, exiting non-zero also aborts the `git am` script without committing the patch.

`git am` 명령에서 마지막으로 실행되는 훅은 `post-applypatch`다. 이 스크립트를 이용하면 자동으로 Patch를 보내준 사람이나 그룹에 알림 메시지를 보낼 수 있다. 이 스크립트로는 Patch를 중단 시킬 수 없다.

The last hook to run during a `git am` operation is `post-applypatch`. You can use it to notify a group or the author of the patch you pulled in that you’ve done so. You can’t stop the patching process with this script.

#### 기타 훅 ####

`pre-rebase` 훅은 Rebase하기 전에 실행되는 것인데 이 훅이 0이 아닌 값을 반환하면 Rebase가 취소된다. 이 훅으로 이미 Push한 커밋을 Rebase하지 못하게 할 수 있는데, Git이 자동으로 넣어주는 `pre-rebase` 예제가 바로 그 예제다. 이 예제에는 기준 브랜치가 `next`라고 돼 있지만 실제로 사용할 브랜치로 변경해서 사용할 수 있다.

The `pre-rebase` hook runs before you rebase anything and can halt the process by exiting non-zero. You can use this hook to disallow rebasing any commits that have already been pushed. The example `pre-rebase` hook that Git installs does this, although it assumes that next is the name of the branch you publish. You’ll likely need to change that to whatever your stable, published branch is.

그리고 `git checkout` 명령이 끝나면 `post-checkout` 훅이 실행된다. 이 훅은 Checkout할 때마다 작업하는 디렉토리에서 뭔가 할 일이 있을 때 사용한다. 그러니까 용량이 크거나 Git이 관리하지 않는 파일을 옮기거나, 문서를 자동으로 생성하는 것 같은 일을 하는 데 쓴다.

After you run a successful `git checkout`, the `post-checkout` hook runs; you can use it to set up your working directory properly for your project environment. This may mean moving in large binary files that you don’t want source controlled, auto-generating documentation, or something along those lines.

마지막으로, `post-merge` 훅은 Merge가 끝난 후에 실행된다. 이 훅은 파일 권한 같이 Git이 추적하지 않는 정보를 관리하는데 사용하거나 Merge로 Working Tree가 변경될 때 Git이 관리하지 않는 파일이 원하는 대로 잘 배치됐는지 검사할 수 있다.

Finally, the `post-merge` hook runs after a successful `merge` command. You can use it to restore data in the working tree that Git can’t track, such as permissions data. This hook can likewise validate the presence of files external to Git control that you may want copied in when the working tree changes.

### 서버 훅 ###

클라이언트 훅으로도 어떤 정책을 강제할 수 있지만, 시스템 관리자에게는 서버 훅이 더 중요하다. 서버 훅은 모두 Push 전후에 실행된다. Push 전에 실행되는 훅이 0이 아닌 값을 반환하면 해당 Push는 거절되고 클라이언트는 에러 메시지를 출력한다. 이 훅으로 아주 복잡한 Push 정책도 설정할 수 있다.

In addition to the client-side hooks, you can use a couple of important server-side hooks as a system administrator to enforce nearly any kind of policy for your project. These scripts run before and after pushes to the server. The pre hooks can exit non-zero at any time to reject the push as well as print an error message back to the client; you can set up a push policy that’s as complex as you wish.

#### pre-receive와 post-receive ####

Push하면 가장 처음 실행되는 훅은 `pre-receive` 훅이다. 이 스크립트는 표준 입력(STDIN)으로 Push하는 참조의 목록을 입력받고 0이 아닌 값을 반환하면 해당 참조가 전부 거절된다. Fast-forward Push가 아니면 거절하거나, 관리자가 브랜치를 새로 Push하고 삭제하는 것을 허용하고 일반 개발자는 수정사항만 Push할 수 있게 하려면 이 훅에서 하는 것이 좋다.

The first script to run when handling a push from a client is `pre-receive`. It takes a list of references that are being pushed from stdin; if it exits non-zero, none of them are accepted. You can use this hook to do things like make sure none of the updated references are non-fast-forwards; or to check that the user doing the pushing has create, delete, or push access or access to push updates to all the files they’re modifying with the push.

`post-receive` 훅은 Push한 후에 실행되고 이 훅으로는 사용자나 서비스에 알림 메시지를 보낼 수 있다. 그리고 `pre-receive` 훅처럼 STDIN으로 참조 목록이 넘어간다. 이 훅으로 메일링리스트에 메일을 보내거나, CI(Continuous Integration) 서버나 Ticket-tracking 시스템의 정보를 수정할 수 있다. 심지어 커밋 메시지도 파싱할 수 있기 때문에 이 훅으로 Ticket을 만들고, 수정하고, 닫을 수 있다. 이 스크립트가 완전히 종료할 때까지 클라이언트와의 연결이 유지되고 Push를 중단시킬 수 없다. 그래서 이 스크립트로 시간이 오래 걸리는 일을 할 때는 조심해야 한다.

The `post-receive` hook runs after the entire process is completed and can be used to update other services or notify users. It takes the same stdin data as the `pre-receive` hook. Examples include e-mailing a list, notifying a continuous integration server, or updating a ticket-tracking system — you can even parse the commit messages to see if any tickets need to be opened, modified, or closed. This script can’t stop the push process, but the client doesn’t disconnect until it has completed; so, be careful when you try to do anything that may take a long time.

#### update ####

update 스크립트는 각 브랜치마다 한 번씩 실행된다는 것을 제외하면 `pre-receive` 스크립트와 거의 같다. 한 번에 브랜치를 여러 개 Push하면 `pre-receive`는 딱 한 번만 실행되지만, update는 브랜치마다 실행된다. 이 스크립트는 표준 입력으로 데이터를 입력받는 것이 아니라 인자로 브랜치 이름, 원래 가리키던 SHA-1 값, 사용자가 Push하는 SHA-1 값을 입력받는다. update 스크립트가 0이 아닌 값을 반환하면 해당 참조만 거절되고 나머지 다른 참조는 상관없다.

The update script is very similar to the `pre-receive` script, except that it’s run once for each branch the pusher is trying to update. If the pusher is trying to push to multiple branches, `pre-receive` runs only once, whereas update runs once per branch they’re pushing to. Instead of reading from stdin, this script takes three arguments: the name of the reference (branch), the SHA-1 that reference pointed to before the push, and the SHA-1 the user is trying to push. If the update script exits non-zero, only that reference is rejected; other references can still be updated.

## 정책 구현하기 ##

지금까지 배운 것을 한 번 적용해보자. 커밋 메시지 규칙 검사하고, Fast-forward Push만 허용하고, 디렉토리마다 사용자의 수정 권한을 제어하는 Git Workflow를 만들어 볼 것이다. 실질적으로 정책을 강제하려면 서버 훅으로 만들어야 하지만 개발자들이 Push할 수 없는 커밋은 아예 만들지 않도록 클라이언트 훅도 만들어 본다. 

In this section, you’ll use what you’ve learned to establish a Git workflow that checks for a custom commit message format, enforces fast-forward-only pushes, and allows only certain users to modify certain subdirectories in a project. You’ll build client scripts that help the developer know if their push will be rejected and server scripts that actually enforce the policies.

나는 내가 제일 좋아하는 Ruby로 만들 것이다. 나는 독자가 슈도코드를 읽듯이 Ruby 코드를 읽을 수 있다고 생각한다. Ruby를 모르더라도 개념을 이해하기엔 충분할 것이다. 하지만 Git은 언어를 가리지 않는다. Git이 자동으로 생성해주는 예제는 모두 Perl과 Bash로 작성돼 있다. 그렇기 때문에 예제를 열어 보면 Perl과 Bash로 작성된 예제를 참고 할 수 있다.

I used Ruby to write these, both because it’s my preferred scripting language and because I feel it’s the most pseudocode-looking of the scripting languages; thus you should be able to roughly follow the code even if you don’t use Ruby. However, any language will work fine. All the sample hook scripts distributed with Git are in either Perl or Bash scripting, so you can also see plenty of examples of hooks in those languages by looking at the samples.

### 서버 훅 ###

서버 정책은 전부 update 훅으로 만든다. 이 스크립트는 브랜치가 Push될 때마다 한 번 실행되고 해당 브랜치의 이름, 원래 브랜치가 가리키던 참조, 새 참조를 인자로 받는다. 그리고 SSH를 통해서 Push하는 것이라면 누가 Push하는지도 알 수 있다. SSH로 접근하긴 하지만 개발자 모두 계정 하나로("git" 같은) Push하고 있다면 실제로 Push하는 사람이 누구인지 식별해주는 쉘 Wrapper가 필요하다. 이 스크립트에서는 `$USER` 환경 변수에 현재 접속한 사용자 정보가 있다고 가정한다. update 스크립트는 필요한 정보를 수집하는 것으로 시작한다:

All the server-side work will go into the update file in your hooks directory. The update file runs once per branch being pushed and takes the reference being pushed to, the old revision where that branch was, and the new revision being pushed. You also have access to the user doing the pushing if the push is being run over SSH. If you’ve allowed everyone to connect with a single user (like "git") via public-key authentication, you may have to give that user a shell wrapper that determines which user is connecting based on the public key, and set an environment variable specifying that user. Here I assume the connecting user is in the `$USER` environment variable, so your update script begins by gathering all the information you need:

	#!/usr/bin/env ruby

	$refname = ARGV[0]
	$oldrev  = ARGV[1]
	$newrev  = ARGV[2]
	$user    = ENV['USER']

	puts "Enforcing Policies... \n(#{$refname}) (#{$oldrev[0,6]}) (#{$newrev[0,6]})"

쉽게 설명하기 위해 전역 변수를 사용했다. 비판하지 말기 바란다.

Yes, I’m using global variables. Don’t judge me — it’s easier to demonstrate in this manner.

#### 커밋 메시지 규칙 만들기 ####

커밋 메시지 규칙부터 해보자. 일단 목표가 있어야 하니까 커밋 메시지에 "ref: 1234" 같은 스트링이 포함돼 있어야 한다고 가정하자. 보통 커밋은 이슈 트래커에 있는 이슈와 관련돼 있으니 그 이슈가 뭔지 커밋메시지에 적으면 좋다. Push할 때마다 모든 커밋 메시지에 해당 문자열이 포함돼 있는지 확인한다. 만약 스트링이 없는 커밋이 있으면 0이 아닌 값을 반환해서 Push를 거절한다.

Your first challenge is to enforce that each commit message must adhere to a particular format. Just to have a target, assume that each message has to include a string that looks like "ref: 1234" because you want each commit to link to a work item in your ticketing system. You must look at each commit being pushed up, see if that string is in the commit message, and, if the string is absent from any of the commits, exit non-zero so the push is rejected.

`$newrev`, `$oldrev` 변수와 `git rev-list`라는 Plumbing 명령어를 이용해서 Push하는 커밋의 모든 SHA-1 값을 알 수 있다. 이것은 `git log`와 근본적으로 같은 명령이고 옵션을 하나도 주지 않으면 다른 정보 없이 SHA-1 값만 보여준다. 이 명령으로 Push하는 커밋이 무엇인지 알아 낼 수 있다:

You can get a list of the SHA-1 values of all the commits that are being pushed by taking the `$newrev` and `$oldrev` values and passing them to a Git plumbing command called `git rev-list`. This is basically the `git log` command, but by default it prints out only the SHA-1 values and no other information. So, to get a list of all the commit SHAs introduced between one commit SHA and another, you can run something like this:

	$ git rev-list 538c33..d14fc7
	d14fc7c847ab946ec39590d87783c69b031bdfb7
	9f585da4401b0a3999e84113824d15245c13f0be
	234071a1be950e2a8d078e6141f5cd20c1e61ad3
	dfa04c9ef3d5197182f13fb5b9b1fb7717d2222a
	17716ec0f1ff5c77eff40b7fe912f9f6cfd0e475

이 SHA-1 값으로 각 커밋의 메시지도 가져온다. 커밋 메시지를 가져와서 정규표현식으로 해당 패턴이 있는지 검사한다.

You can take that output, loop through each of those commit SHAs, grab the message for it, and test that message against a regular expression that looks for a pattern.

커밋 메시지를 얻는 방법을 알아보자. 커밋의 raw 데이터는 `git cat-file`이라는 Plumbing 명령어로 얻을 수 있다. 9장에서 Plumbing 명령어에 대해 자세히 다루니까 지금은 커밋 메시지 얻는 것에 집중하자:

You have to figure out how to get the commit message from each of these commits to test. To get the raw commit data, you can use another plumbing command called `git cat-file`. I’ll go over all these plumbing commands in detail in Chapter 9; but for now, here’s what that command gives you:

	$ git cat-file commit ca82a6
	tree cfda3bf379e4f8dba8717dee55aab78aef7f4daf
	parent 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	author Scott Chacon <schacon@gmail.com> 1205815931 -0700
	committer Scott Chacon <schacon@gmail.com> 1240030591 -0700

	changed the version number

이 명령이 출력하는 메시지에서 커밋 메시지만 잘라내야 한다. 첫번째 빈 줄 다음부터가 커밋 메시지니까 유닉스 명령어 `sed`로 첫 빈 줄 이후를 잘라낸다.

A simple way to get the commit message from a commit when you have the SHA-1 value is to go to the first blank line and take everything after that. You can do so with the `sed` command on Unix systems:

	$ git cat-file commit ca82a6 | sed '1,/^$/d'
	changed the version number

이제 커밋 메시지에서 찾는 패턴과 일치하는 문자열이 있는지 검사해서 있으면 통과시키고 없으면 거절한다. 스크립트가 종료할 때 0이 아닌 값을 반환하면 Push가 거절된다. 이 일을 하는 코드는 다음과 같다:

You can use that incantation to grab the commit message from each commit that is trying to be pushed and exit if you see anything that doesn’t match. To exit the script and reject the push, exit non-zero. The whole method looks like this:

	$regex = /\[ref: (\d+)\]/

	# enforced custom commit message format
	def check_message_format
	  missed_revs = `git rev-list #{$oldrev}..#{$newrev}`.split("\n")
	  missed_revs.each do |rev|
	    message = `git cat-file commit #{rev} | sed '1,/^$/d'`
	    if !$regex.match(message)
	      puts "[POLICY] Your message is not formatted correctly"
	      exit 1
	    end
	  end
	end
	check_message_format

이 코드를 update 스크립트에 넣으면 규칙을 어긴 커밋은 Push할 수 없다.

Putting that in your update script will reject updates that contain commits that have messages that don’t adhere to your rule.

#### ACL로 사용자마다 다른 규칙 적용하기 ####

진행하는 프로젝트에 모듈이 여러개 있는데, 각 모듈마다 속한 사용자들만 Push할 수 있게 설정해야 한다고 가정하자. 모든 권한을 다 가진 사람들도 있고 특정 디렉토리나 파일만 푸시할 수 있는 사람들도 있을 것이다. 이런 일을 하려면 먼저 서버의 Bare 저장소에 `acl` 이라는 파일을 만들고 거기에 규칙을 기술한다. 그리고 update 훅에서 Push하는 파일이 무엇인지 확인하고 ACL과 비교해서 Push할 수 있는지 없는지 결정한다.

Suppose you want to add a mechanism that uses an access control list (ACL) that specifies which users are allowed to push changes to which parts of your projects. Some people have full access, and others only have access to push changes to certain subdirectories or specific files. To enforce this, you’ll write those rules to a file named `acl` that lives in your bare Git repository on the server. You’ll have the update hook look at those rules, see what files are being introduced for all the commits being pushed, and determine whether the user doing the push has access to update all those files.

우선 ACL부터 작성한다. CVS에서 사용하는 것과 비슷한 ACL을 만들어 사용할 것이다. 규칙은 한 줄에 하나씩 기술한다. 각 줄의 첫번째 필드는 `avail`이나 `unavail`이고 두 번째 필드는 규칙을 적용할 사용자들의 목록을 CSV(Comma-Separated Values) 형식으로 적는다. 마지막 필드엔 규칙을 적용할 경로를 적는다. 만약 마지막 필드가 비워져 있으면 모든 경로를 의미한다. 이 필드들은 파이프(`|`) 문자로 구분한다.

The first thing you’ll do is write your ACL. Here you’ll use a format very much like the CVS ACL mechanism: it uses a series of lines, where the first field is `avail` or `unavail`, the next field is a comma-delimited list of the users to which the rule applies, and the last field is the path to which the rule applies (blank meaning open access). All of these fields are delimited by a pipe (`|`) character.

관리자도 여러명이고, `doc` 디렉토리에서 문서를 만드는 사람도 여려명이고, `lib`과 `tests` 디렉토리에 접근하는 사람은 한명이다. 이런 상황을 ACL로 만들면 다음과 같다:

In this case, you have a couple of administrators, some documentation writers with access to the `doc` directory, and one developer who only has access to the `lib` and `tests` directories, and your ACL file looks like this:

	avail|nickh,pjhyett,defunkt,tpw
	avail|usinclair,cdickens,ebronte|doc
	avail|schacon|lib
	avail|schacon|tests

이 ACL 정보는 스크립트에서 읽어 사용한다. 설명을 쉽게 하고자 여기서는 `avail`만 처리한다. 다음 메소드는 Associative Array를 반환하는데, 키는 사용자이름이고 값은 사용자가 Push할 수 있는 경로의 목록이다:

You begin by reading this data into a structure that you can use. In this case, to keep the example simple, you’ll only enforce the `avail` directives. Here is a method that gives you an associative array where the key is the user name and the value is an array of paths to which the user has write access:

	def get_acl_access_data(acl_file)
	  # read in ACL data
	  acl_file = File.read(acl_file).split("\n").reject { |line| line == '' }
	  access = {}
	  acl_file.each do |line|
	    avail, users, path = line.split('|')
	    next unless avail == 'avail'
	    users.split(',').each do |user|
	      access[user] ||= []
	      access[user] << path
	    end
	  end
	  access
	end

이 함수가 ACL 파일을 처리하고 나서 반환하는 결과는 다음과 같다:

On the ACL file you looked at earlier, this `get_acl_access_data` method returns a data structure that looks like this:

	{"defunkt"=>[nil],
	 "tpw"=>[nil],
	 "nickh"=>[nil],
	 "pjhyett"=>[nil],
	 "schacon"=>["lib", "tests"],
	 "cdickens"=>["doc"],
	 "usinclair"=>["doc"],
	 "ebronte"=>["doc"]}

바로 사용할 수 있는 권한 정보를 만들었다. 이제 Push하는 커밋에 있는 파일을 그 사용자가 Push할 수 있는지 없는지 알아내야 한다.

Now that you have the permissions sorted out, you need to determine what paths the commits being pushed have modified, so you can make sure the user who’s pushing has access to all of them.

`git log` 명령에 `--name-only` 옵션을 주면 해당 커밋에서 수정된 파일이 뭔지 알려준다. `git log` 명령은 2 장에서 다뤘었다:

You can pretty easily see what files have been modified in a single commit with the `--name-only` option to the `git log` command (mentioned briefly in Chapter 2):

	$ git log -1 --name-only --pretty=format:'' 9f585d

	README
	lib/test.rb

`get_acl_access_data` 메소드를 호출해서 ACL 정보를 구하고, 각 커밋에 들어 있는 파일 목록도 얻은 다음에, 사용자가 모든 커밋을 Push할 수 있는지 판단한다:

If you use the ACL structure returned from the `get_acl_access_data` method and check it against the listed files in each of the commits, you can determine whether the user has access to push all of their commits:

	# only allows certain users to modify certain subdirectories in a project
	def check_directory_perms
	  access = get_acl_access_data('acl')

	  # see if anyone is trying to push something they can't
	  new_commits = `git rev-list #{$oldrev}..#{$newrev}`.split("\n")
	  new_commits.each do |rev|
	    files_modified = `git log -1 --name-only --pretty=format:'' #{rev}`.split("\n")
	    files_modified.each do |path|
	      next if path.size == 0
	      has_file_access = false
	      access[$user].each do |access_path|
	        if !access_path  # user has access to everything
	          || (path.index(access_path) == 0) # access to this path
	          has_file_access = true
	        end
	      end
	      if !has_file_access
	        puts "[POLICY] You do not have access to push to #{path}"
	        exit 1
	      end
	    end
	  end
	end

	check_directory_perms

따라오기 어렵지 않을 것이다. 먼저 `git rev-list` 명령으로 서버에 Push하려는 커밋이 무엇인지 알아낸다. 그리고 각 커밋에서 수정한 파일이 어떤 것들이 있는지 찾고, 해당 사용자가 모든 파일에 권한을 가지고 있는지 확인한다. Rubyism 철학에 따르면 `path.index(access_path) == 0`이란 표현은 불명확하다. 이 표현은 해당 파일의 경로가 `access_path`로 시작할 때 참이라는 뜻이다. 그러니까 `access_path`가 단순히 허용된 파일 하나를 의미하는 것이 아니라 `access_path`로 시작하는 모든 파일을 의미하는 것이다.

Most of that should be easy to follow. You get a list of new commits being pushed to your server with `git rev-list`. Then, for each of those, you find which files are modified and make sure the user who’s pushing has access to all the paths being modified. One Rubyism that may not be clear is `path.index(access_path) == 0`, which is true if path begins with `access_path` — this ensures that `access_path` is not just in one of the allowed paths, but an allowed path begins with each accessed path. 

이제 사용자는 메시지 규칙을 어겼거나 권한이 없는 파일을 수정한 커밋은 어떤 것도 Push하지 못한다.

Now your users can’t push any commits with badly formed messages or with modified files outside of their designated paths.

#### Fast-Forward Push만 허용하기 ####

이제 Fast-forward Push가 아니면 거절해보자. Git 1.6 부터 `receive.denyDeletes`와 `receive.denyNonFastForwards` 설정으로 간단하게 사용할 수도 있다. 하지만 그 이전 버전에는 꼭 훅으로 구현해야 했다. 게다가 특정 사용자만 제한하거나 허용하는 것을 하려면 훅으로 구현해야 한다.

The only thing left is to enforce fast-forward-only pushes. In Git versions 1.6 or newer, you can set the `receive.denyDeletes` and `receive.denyNonFastForwards` settings. But enforcing this with a hook will work in older versions of Git, and you can modify it to do so only for certain users or whatever else you come up with later.

새로 푸시하는 커밋에서는 찾을 수 없고(aren't reachable) 예전 커밋에서만 찾을 수 있는 커밋이 있는지 확인하면 Fast-forward Push인지를 검사할 수 있다. 하나라도 있으면 거절하고 없으면 Fast-forward Push임으로 그대로 둔다:

The logic for checking this is to see if any commits are reachable from the older revision that aren’t reachable from the newer one. If there are none, then it was a fast-forward push; otherwise, you deny it:

	# enforces fast-forward only pushes 
	def check_fast_forward
	  missed_refs = `git rev-list #{$newrev}..#{$oldrev}`
	  missed_ref_count = missed_refs.split("\n").size
	  if missed_ref_count > 0
	    puts "[POLICY] Cannot push a non fast-forward reference"
	    exit 1
	  end
	end

	check_fast_forward

이 정책을 다 구현해서 update 스크립트에 넣고 `chmod u+x .git/hooks/update` 명령으로 실행 권한을 준다. 그리고 나서 `-f` 옵션을 주고 강제로 Push하면 다음과 같이 실패할 것이다:

Everything is set up. If you run `chmod u+x .git/hooks/update`, which is the file you into which you should have put all this code, and then try to push a non-fast-forwarded reference, you get something like this:

	$ git push -f origin master
	Counting objects: 5, done.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 323 bytes, done.
	Total 3 (delta 1), reused 0 (delta 0)
	Unpacking objects: 100% (3/3), done.
	Enforcing Policies... 
	(refs/heads/master) (8338c5) (c5b616)
	[POLICY] Cannot push a non-fast-forward reference
	error: hooks/update exited with error code 1
	error: hook declined to update refs/heads/master
	To git@gitserver:project.git
	 ! [remote rejected] master -> master (hook declined)
	error: failed to push some refs to 'git@gitserver:project.git'

정책과 관련해 하나씩 살펴보자. 먼저 훅이 실행될 때마다 다음 메시지가 출력된다.

There are a couple of interesting things here. First, you see this where the hook starts running.

	Enforcing Policies... 
	(refs/heads/master) (fb8c72) (c56860)

이것은 update 스크립트 맨 윗부분에서 표준출력(STDOUT)으로 출력한 내용이다. 스크립트에서 표준출력으로 출력하면 클라이언트로 전송된다는 것을 꼭 기억하자.

Notice that you printed that out to stdout at the very beginning of your update script. It’s important to note that anything your script prints to stdout will be transferred to the client.

그리고 다음 에러 메시지를 보자:

The next thing you’ll notice is the error message.

	[POLICY] Cannot push a non fast-forward reference
	error: hooks/update exited with error code 1
	error: hook declined to update refs/heads/master

첫 번째 줄은 스크립트에서 직접 출력한 것이고 나머지 두 2줄은 Git이 출력해 주는 것이다. 이 메시지는 update 스크립트에서 0이 아닌 값이 반환했기 때문에 Push할 수 없다고 말하는 것이다. 그리고 마지막 메시지를 보자:

The first line was printed out by you, the other two were Git telling you that the update script exited non-zero and that is what is declining your push. Lastly, you have this:

	To git@gitserver:project.git
	 ! [remote rejected] master -> master (hook declined)
	error: failed to push some refs to 'git@gitserver:project.git'

이 메시지는 훅에서 거절된 것이라고 말해주는 것이고 브랜치가 거부될 때마다 하나씩 출력된다.

You’ll see a remote rejected message for each reference that your hook declined, and it tells you that it was declined specifically because of a hook failure.

게다가 Push하는 커밋에 커밋 메시지 규칙을 지키지 않은 것이 하나라도 있으면 다음과 같은 에러 메시지를 보여준다:

Furthermore, if the ref marker isn’t there in any of your commits, you’ll see the error message you’re printing out for that.

	[POLICY] Your message is not formatted correctly

그리고 누군가 권한이 없는 파일을 수정해서 Push하면 에러 메시지를 출력한다. 예를 들어 문서 담당자가 `lib` 디렉토리에 있는 파일을 수정해서 커밋하면 다음과 같은 메시지가 출력된다:

Or if someone tries to edit a file they don’t have access to and push a commit containing it, they will see something similar. For instance, if a documentation author tries to push a commit modifying something in the `lib` directory, they see

	[POLICY] You do not have access to push to lib/test.rb

이제 서버 훅은 다 했다. 앞으로 update 스크립트가 항상 실행될 것이기 때문에 저장소를 되감을 수 없고, 커밋 메시지도 규칙대로 작성해야 하고, 권한이 있는 파일만 Push할 수 있다.

That’s all. From now on, as long as that update script is there and executable, your repository will never be rewound and will never have a commit message without your pattern in it, and your users will be sandboxed.

### 클라이언트 훅 ###

서버 훅의 단점은 Push할 때까지 Push할 수 있는지 없는지 알 수 없다는 것이다. 기껏 공들여 정성껏 구현했는데 막상 Push할 수 없으면 곤혹스러울 것이다. 게다가 히스토리를 제대로 고치는 일은 정신건강에 해롭다. 

The downside to this approach is the whining that will inevitably result when your users’ commit pushes are rejected. Having their carefully crafted work rejected at the last minute can be extremely frustrating and confusing; and furthermore, they will have to edit their history to correct it, which isn’t always for the faint of heart.

이 문제는 클라이언트 훅으로 해결할 수 있다. 사용자는 클라이언트 훅으로 서버가 거부할지 말지 검사할 수 있다. 즉 사람들은 커밋하기 전에, 그러니까 시간이 지나 고치기 어려워지기 전에 문제를 해결할 수 있다. Clone할 때 이 훅은 전송되지 않기 때문에 다른 방법으로 동료에게 배포해야 한다. 그 훅을 가져다 `.git/hooks` 디렉토리에 복사하고 실행할 수 있게 만든다. 이 훅 파일을 프로젝트에 넣어서 배포해도 되고 전용 Git 프로젝트를 만들어서 배포해도 된다. 하지만 자동으로 설치되도록 할 방법은 없다.

The answer to this dilemma is to provide some client-side hooks that users can use to notify them when they’re doing something that the server is likely to reject. That way, they can correct any problems before committing and before those issues become more difficult to fix. Because hooks aren’t transferred with a clone of a project, you must distribute these scripts some other way and then have your users copy them to their `.git/hooks` directory and make them executable. You can distribute these hooks within the project or in a separate project, but there is no way to set them up automatically.

커밋 메시지부터 검사해보자. 이 훅이 있으면 나중에 커밋 메시지가 구리다고 서버가 거절하지 않을 것이다. 이것은 `commit-msg` 훅으로 구현한다. 이 훅은 첫번째 인자로 커밋 메시지가 저장된 파일을 입력 받는다. 그 파일을 읽어 패턴을 검사한다. 필요한 패턴이 없으면 커밋을 중단 시킨다:

To begin, you should check your commit message just before each commit is recorded, so you know the server won’t reject your changes due to badly formatted commit messages. To do this, you can add the `commit-msg` hook. If you have it read the message from the file passed as the first argument and compare that to the pattern, you can force Git to abort the commit if there is no match:

	#!/usr/bin/env ruby
	message_file = ARGV[0]
	message = File.read(message_file)

	$regex = /\[ref: (\d+)\]/

	if !$regex.match(message)
	  puts "[POLICY] Your message is not formatted correctly"
	  exit 1
	end

이 스크립트를 `.git/hooks/commit-msg`라는 파일로 만들고 실행권한을 준다. 커밋이 메시지 규칙을 어기면 다음과 같은 메시지를 보여 준다:

If that script is in place (in `.git/hooks/commit-msg`) and executable, and you commit with a message that isn’t properly formatted, you see this:

	$ git commit -am 'test'
	[POLICY] Your message is not formatted correctly

커밋하지 못했다. 하지만 커밋 메지시가 바르게 작성되면 커밋할 수 있다:

No commit was completed in that instance. However, if your message contains the proper pattern, Git allows you to commit:

	$ git commit -am 'test [ref: 132]'
	[master e05c914] test [ref: 132]
	 1 files changed, 1 insertions(+), 0 deletions(-)

그리고 아예 권한이 없는 파일을 수정할 수 없게 하려면 `pre-commit` 훅을 이용한다. 사전에 `.git` 디렉토리 안에 ACL 파일을 가져다 놓고 다음과 같이 작성한다:

Next, you want to make sure you aren’t modifying files that are outside your ACL scope. If your project’s `.git` directory contains a copy of the ACL file you used previously, then the following `pre-commit` script will enforce those constraints for you:

	#!/usr/bin/env ruby

	$user    = ENV['USER']

	# [ insert acl_access_data method from above ]

	# only allows certain users to modify certain subdirectories in a project
	def check_directory_perms
	  access = get_acl_access_data('.git/acl')

	  files_modified = `git diff-index --cached --name-only HEAD`.split("\n")
	  files_modified.each do |path|
	    next if path.size == 0
	    has_file_access = false
	    access[$user].each do |access_path|
	    if !access_path || (path.index(access_path) == 0)
	      has_file_access = true
	    end
	    if !has_file_access
	      puts "[POLICY] You do not have access to push to #{path}"
	      exit 1
	    end
	  end
	end

	check_directory_perms

내용은 서버 훅과 똑같지만 두 가지가 다르다. 첫째, 클라이언트 훅은 Git 디렉토리가 아니라 Working Directory에서 실행하기 때문에 ACL 파일 위치가 다르다. 그래서 ACL 파일 경로를 수정해야 한다:

This is roughly the same script as the server-side part, but with two important differences. First, the ACL file is in a different place, because this script runs from your working directory, not from your Git directory. You have to change the path to the ACL file from this

	access = get_acl_access_data('acl')

이 부분을 다음과 같이 바꾼다:

to this:

	access = get_acl_access_data('.git/acl')

두 번째 차이점은 파일 목록을 얻는 방법이다. 서버 훅에서는 커밋에 있는 파일을 모두 찾았지만 여기서는 아직 커밋하지도 않았다. 그래서 Stage 영역의 파일 목록을 이용한다:

The other important difference is the way you get a listing of the files that have been changed. Because the server-side method looks at the log of commits, and, at this point, the commit hasn’t been recorded yet, you must get your file listing from the staging area instead. Instead of

	files_modified = `git log -1 --name-only --pretty=format:'' #{ref}`

이 부분을 다음과 같이 바꾼다:

you have to use

	files_modified = `git diff-index --cached --name-only HEAD`

이 두 가지 점만 다르고 나머지는 똑같다. 보통은 원격 저장소의 계정과 로컬의 계정도 같다. 하지만 다른 계정을 사용한다면 `$user` 환경변수에 누군지 알려야 한다.

But those are the only two differences — otherwise, the script works the same way. One caveat is that it expects you to be running locally as the same user you push as to the remote machine. If that is different, you must set the `$user` variable manually.

Fast-forward Push인지 확인하는 일이 남았다. 보통은 Fast-forward가 아닌 Push는 그 자체가 드문 일이다. Fast-forward가 아닌 Push를 하려면 Rebase로 이미 Push한 커밋을 바꿔 버렸거나 전혀 다른 로컬 브랜치를 Push해야 한다

The last thing you have to do is check that you’re not trying to push non-fast-forwarded references, but that is a bit less common. To get a reference that isn’t a fast-forward, you either have to rebase past a commit you’ve already pushed up or try pushing a different local branch up to the same remote branch.

어쨌든 이 서버는 Fast-forward Push만 허용하기 때문에 이미 Push한 커밋을 수정했다면 그건 아마 실수일 것이다. 이 실수를 막는 훅을 살펴보자.

Because the server will tell you that you can’t push a non-fast-forward anyway, and the hook prevents forced pushes, the only accidental thing you can try to catch is rebasing commits that have already been pushed.

다음은 이미 Push한 커밋을 Rebase하지 못하게 하는 pre-rebase 스크립트다. 이 스크립트는 대상 커밋 목록을 얻어서 원격 참조/브랜치에 들어 있는지 확인한다. 커밋이 한 개라도 원격 브랜치/참조에 들어 있으면 Rebase할 수 없다:

Here is an example pre-rebase script that checks for that. It gets a list of all the commits you’re about to rewrite and checks whether they exist in any of your remote references. If it sees one that is reachable from one of your remote references, it aborts the rebase:

	#!/usr/bin/env ruby

	base_branch = ARGV[0]
	if ARGV[1]
	  topic_branch = ARGV[1]
	else
	  topic_branch = "HEAD"
	end

	target_shas = `git rev-list #{base_branch}..#{topic_branch}`.split("\n")
	remote_refs = `git branch -r`.split("\n").map { |r| r.strip }

	target_shas.each do |sha|
	  remote_refs.each do |remote_ref|
	    shas_pushed = `git rev-list ^#{sha}^@ refs/remotes/#{remote_ref}`
	    if shas_pushed.split(“\n”).include?(sha)
	      puts "[POLICY] Commit #{sha} has already been pushed to #{remote_ref}"
	      exit 1
	    end
	  end
	end

이 스크립트는 6장 '리비전 조회하기' 절에서 설명하지 않은 표현을 사용한다. 이미 Push한 커밋 목록을 얻어오는 부분은 다음과 같다:

This script uses a syntax that wasn’t covered in the Revision Selection section of Chapter 6. You get a list of commits that have already been pushed up by running this:

	git rev-list ^#{sha}^@ refs/remotes/#{remote_ref}

`SHA^@`은 해당 커밋의 모든 부모를 가리킨다. 그러니까 이 명령은 지금 Push하려는 커밋에서 원격 저장소의 커밋에 도달 할 수 있는지 확인하는 것이다. 즉, Fast-forward인지 확인하는 것이다.

The `SHA^@` syntax resolves to all the parents of that commit. You’re looking for any commit that is reachable from the last commit on the remote and that isn’t reachable from any parent of any of the SHAs you’re trying to push up — meaning it’s a fast-forward.

이 방법의 문제는 매우 느리고 보통은 필요 없다는 것이다. 어짜피 Fast-forward가 아닌 Push은 `-f` 옵션을 주어야 Push할 수 있다. 하지만 이 예제는 이론적으로 문제가 될만한 Rebase를 방지할 있다는 것을 보여준다.

The main drawback to this approach is that it can be very slow and is often unnecessary — if you don’t try to force the push with `-f`, the server will warn you and not accept the push. However, it’s an interesting exercise and can in theory help you avoid a rebase that you might later have to go back and fix.

## 요약 ##

프로젝트에 적절하도록 Git을 설정하는 방법을 배웠다. 주요한 서버/클라이언트 설정 방법, 파일 단위로 설정하는 Attributes, 이벤트 훅, 정책을 강제하는 방법을 배웠다. 이제 필요한 Workflow를 만들고 Git을 거기에 맞게 설정할 수 있을 것이다.

You’ve covered most of the major ways that you can customize your Git client and server to best fit your workflow and projects. You’ve learned about all sorts of configuration settings, file-based attributes, and event hooks, and you’ve built an example policy-enforcing server. You should now be able to make Git fit nearly any workflow you can dream up.

