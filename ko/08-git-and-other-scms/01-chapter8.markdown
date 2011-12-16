# Git and Other Systems / Git과 타 시스템 #

The world isn’t perfect. Usually, you can’t immediately switch every project you come in contact with to Git. Sometimes you’re stuck on a project using another VCS, and many times that system is Subversion. You’ll spend the first part of this chapter learning about `git svn`, the bidirectional Subversion gateway tool in Git.

세상은 완벽하지 않다. 보통 대부분의 프로젝트를 Git으로 옮겨오는 것은 쉽지 않다. 프로젝트가 다른 VCS 시스템에 매우 단단히 결합되어 있을 수 있으며, 보통 Subversion인 경우가 많다. 이번 장은 `git svn` 이라는 양방향으로 Subversion을 이용할 수 있는 도구를 알아보며 시작한다.

At some point, you may want to convert your existing project to Git. The second part of this chapter covers how to migrate your project into Git: first from Subversion, then from Perforce, and finally via a custom import script for a nonstandard importing case. 

언젠가 이미 존재하는 프로젝트 환경을 Git을 사용하는 환경으로 변경하고 싶게 될지도 모른다. 이 장의 나머지 부분에서 Git으로 프로젝트를 변경하는 방법에 대해 다룰 것이다. Subversion부터 시작해서 Perforce 그리고 사용자만의 스크립트를 만들어서 널리 쓰이지 않는 VCS로부터 프로젝트를 옮겨오는 방법을 다룰 것이다.

## Git and Subversion / Git과 Subversion ##

Currently, the majority of open source development projects and a large number of corporate projects use Subversion to manage their source code. It’s the most popular open source VCS and has been around for nearly a decade. It’s also very similar in many ways to CVS, which was the big boy of the source-control world before that.

현재 주요 오픈소스 프로젝트와 아주 많은 수의 기업 프로젝트에서 소스코드 관리를 위해 Subversion을 사용한다. 10여년간 Subversion은 가장 인기있는 오픈소스 VCS 도구였다. 많은 부분에서 이전 시대에서 가장 많이 사용하였던 CVS와 많이 닮았다.

One of Git’s great features is a bidirectional bridge to Subversion called `git svn`. This tool allows you to use Git as a valid client to a Subversion server, so you can use all the local features of Git and then push to a Subversion server as if you were using Subversion locally. This means you can do local branching and merging, use the staging area, use rebasing and cherry-picking, and so on, while your collaborators continue to work in their dark and ancient ways. It’s a good way to sneak Git into the corporate environment and help your fellow developers become more efficient while you lobby to get the infrastructure changed to support Git fully. The Subversion bridge is the gateway drug to the DVCS world.

Git이 자랑하는 또 하나의 기능은 `git svn`이라는 양방향 Subversion 지원 도구이다. Git을 Subversion 서버를 사용하는 제대로 된 기능의 클라이언트로 사용할 수 있기 때문에 로컬에서는 Git의 기능을 활용하고 Push 할 때 Subversion 서버에 Push 할 수 있다. 즉 로컬 브랜치와 Merge, Stage 영역, Rebase, Chrry-pick 등의 Git 기능을 충분히 사용할 수 있다. 같이 일하는 동료는 선사시대 빛도 없는 곳에서 일하겠지만 말이다. `git svn`은 기업의 개발 환경에서 git을 사용하는 출발점으로 사용할 수 있고 우리가 Git을 도입하기 위해 기업내에서 노력하는 동안 동료가 효율적으로 환경을 바꿀 수 있도록 도움을 줄 수 잇다. Subversion 지원 도구는 DVCS 세상으로 인도하는 붉은 알약과 같은 것이다.

### git svn ###

The base command in Git for all the Subversion bridging commands is `git svn`. You preface everything with that. It takes quite a few commands, so you’ll learn about the common ones while going through a few small workflows.

Git과 Subversion을 이어주는 명령은 `git svn` 으로 시작한다. 이 명령 뒤에 추가적으로 몇 가지 더 명령이 정의되어 있으며 이어지는 작은 예제들을 통해 이해하도록 한다.

It’s important to note that when you’re using `git svn`, you’re interacting with Subversion, which is a system that is far less sophisticated than Git. Although you can easily do local branching and merging, it’s generally best to keep your history as linear as possible by rebasing your work and avoiding doing things like simultaneously interacting with a Git remote repository.

`git svn` 명령을 사용할 때는 턱없이 모자란 Subversion이 함께 동작한다는 점을 염두해두자. 우리가 로컬 브랜치와 머지 기능을 손쉽게 쓸 수 있다고 하더라도 최대한 일직선으로 히스토리를 유지하는것이 좋다. Git 저장소를 사용하는것 처럼 하지 않는 것이 좋다.

Don’t rewrite your history and try to push again, and don’t push to a parallel Git repository to collaborate with fellow Git developers at the same time. Subversion can have only a single linear history, and confusing it is very easy. If you’re working with a team, and some are using SVN and others are using Git, make sure everyone is using the SVN server to collaborate — doing so will make your life easier.

히스토리를 재작성하지 말아야 하고 Push를 재전송하지도 말아야 한다. 동시에 같은 Git 저장소에 Push하지도 말아야 한다. Subversion은 단순히 일직선의 히스토리만 가질 수 있다. 우리가 일부는 SVN을 일부는 Git을 사용하는 팀에 있을 때에는 협업을 위해서 모두가 SVN Server를 사용해야 한다. 그래야 삶이 편하다.

### Setting Up / 설정하기 ###

To demonstrate this functionality, you need a typical SVN repository that you have write access to. If you want to copy these examples, you’ll have to make a writeable copy of my test repository. In order to do that easily, you can use a tool called `svnsync` that comes with more recent versions of Subversion — it should be distributed with at least 1.4. For these tests, I created a new Subversion repository on Google code that was a partial copy of the `protobuf` project, which is a tool that encodes structured data for network transmission. 

이 기능을 써보기이 위해 우리는 SVN 저장소 하나가 필요하다. 물론 쓰기 권한도 있어야 한다. 아래 나오는 예제를 써보려면 저자의 test 저장소를 하나 복사해야 한다. SVN 저장소를 복사하기 위해 최근의 Subversion(1.4 이상) 에 포함된 `svnsync`라는 도구를 사용할 수 있다. 테스트를 해보기 위해 저자는 Google Code에 새로 Subversion 저장소를 하나 만들었고 `protobuf` 라는 프로젝트의 일부 코드를 복사했다. `protobuf`는 네트워크 전송을 위한 구조화된 데이터(프로토콜 같은 것들)의 인코딩을 도와주는 도구이다.

To follow along, you first need to create a new local Subversion repository:

우선 로컬 Subversion 저장소를 하나 만들어야 한다.

	$ mkdir /tmp/test-svn
	$ svnadmin create /tmp/test-svn

Then, enable all users to change revprops — the easy way is to add a pre-revprop-change script that always exits 0:

그 다음, 모든 사용자가 revprops 속성을 변경할 수 있도록 pre-revprop-change 스크립트가 언제나 0을 반환하도록 변경한다. (역주: 파일이 없거나, 다른 이름으로 되어있을 수 있다. 이 경우 아래 내용으로 새로 파일을 만들고 실행 권한을 준다.)

	$ cat /tmp/test-svn/hooks/pre-revprop-change 
	#!/bin/sh
	exit 0;
	$ chmod +x /tmp/test-svn/hooks/pre-revprop-change

You can now sync this project to your local machine by calling `svnsync init` with the to and from repositories.

이제 `svnsync init` 명령으로 다른 Subversion 저장소를 로컬로 복사하도록 지정할 수 있다.

	$ svnsync init file:///tmp/test-svn http://progit-example.googlecode.com/svn/ 

This sets up the properties to run the sync. You can then clone the code by running

위와 같이 다른 저장소의 주소를 설정함으로서 복사할 준비가 되었으며 아래 명령으로 저장소를 실제로 복사할 수 있다.

	$ svnsync sync file:///tmp/test-svn
	Committed revision 1.
	Copied properties for revision 1.
	Committed revision 2.
	Copied properties for revision 2.
	Committed revision 3.
	...

Although this operation may take only a few minutes, if you try to copy the original repository to another remote repository instead of a local one, the process will take nearly an hour, even though there are fewer than 100 commits. Subversion has to clone one revision at a time and then push it back into another repository — it’s ridiculously inefficient, but it’s the only easy way to do this.

위 명령을 실행했을 때 몇 분 걸리지 않았더라도 저장하는 위치를 로컬이 아니라 원격 서버로 설정하면 이 과정은 엄청 시간이 걸릴것이다. 커밋이 100개 이하의 적은 경우라도 말이다. Subversion이 저장소를 복사하는 과정은 한번에 하나의 커밋을 받아서 Push하기 때문이며 엄청나게 비효율적이지만 저장소를 복사하는 유일한 방법이다.

### Getting Started / 시작하기 ###

Now that you have a Subversion repository to which you have write access, you can go through a typical workflow. You’ll start with the `git svn clone` command, which imports an entire Subversion repository into a local Git repository. Remember that if you’re importing from a real hosted Subversion repository, you should replace the `file:///tmp/test-svn` here with the URL of your Subversion repository:

이제 갖고 놀 Subversion 저장소가 하나 준비되었다. `git svn clone` 명령으로 Subversion 저장소 전체를 Git 저장소로 가져올 수 있다. 만약 로컬의 Subversion 저장소에서 가져오는것이 아니라 서버의 Subversion 저장소에서 가져오려면 `file:///tmp/test-svn` 부분에 서버 저장소의 URL을 적어주어도 된다.

	$ git svn clone file:///tmp/test-svn -T trunk -b branches -t tags
	Initialized empty Git repository in /Users/schacon/projects/testsvnsync/svn/.git/
	r1 = b4e387bc68740b5af56c2a5faf4003ae42bd135c (trunk)
	      A    m4/acx_pthread.m4
	      A    m4/stl_hash.m4
	...
	r75 = d1957f3b307922124eec6314e15bcda59e3d9610 (trunk)
	Found possible branch point: file:///tmp/test-svn/trunk => \
	    file:///tmp/test-svn /branches/my-calc-branch, 75
	Found branch parent: (my-calc-branch) d1957f3b307922124eec6314e15bcda59e3d9610
	Following parent with do_switch
	Successfully followed parent
	r76 = 8624824ecc0badd73f40ea2f01fce51894189b01 (my-calc-branch)
	Checked out HEAD:
	 file:///tmp/test-svn/branches/my-calc-branch r76

This runs the equivalent of two commands — `git svn init` followed by `git svn fetch` — on the URL you provide. This can take a while. The test project has only about 75 commits and the codebase isn’t that big, so it takes just a few minutes. However, Git has to check out each version, one at a time, and commit it individually. For a project with hundreds or thousands of commits, this can literally take hours or even days to finish.

위 명령은 사실 `git svn init`과 SVN 저장소 주소를 지정하여 `git svn fetch` 두 명령을 순서대로 실행한 것과 같다. 명령이 끝나기에 제법 시간이 소요된다. 테스트로 사용하는 프로젝트가 75개 정도의 커밋만 가지고 있기 때문에 그렇게 시간이 많이 걸리지는 않는다. 하지만 한번은 Git이 커밋 하나하나 일일이 기록을 해야 하기 때문에 수천 커밋을 가진 어느정도 규모있는 프로젝트에서는 몇 시간 혹은 몇 일이 걸릴수도 있다.

The `-T trunk -b branches -t tags` part tells Git that this Subversion repository follows the basic branching and tagging conventions. If you name your trunk, branches, or tags differently, you can change these options. Because this is so common, you can replace this entire part with `-s`, which means standard layout and implies all those options. The following command is equivalent:

`-T trunk -b branches -t tags` 부분은 Git에게 Subversion이 어떤 브랜치 구조를 가지고 있는지 정보를 알려주는 부분이다. Subversion의 표준 형식과 다른 이름을 가지고 있다면 이 옵션 부분에서 알맞은 이름을 지정해줄 수 있다. 표준 형식을 사용한다면 간단하게 `-s` 옵션을 사용할 수 있다. 즉 아래의 명령도 같은 의미이다.

	$ git svn clone file:///tmp/test-svn -s

At this point, you should have a valid Git repository that has imported your branches and tags:

브랜치와 태그 정보가 Git에서도 제대로 적용된 모습을 확인할 수 있다:

	$ git branch -a
	* master
	  my-calc-branch
	  tags/2.0.2
	  tags/release-2.0.1
	  tags/release-2.0.2
	  tags/release-2.0.2rc1
	  trunk

It’s important to note how this tool namespaces your remote references differently. When you’re cloning a normal Git repository, you get all the branches on that remote server available locally as something like `origin/[branch]` - namespaced by the name of the remote. However, `git svn` assumes that you won’t have multiple remotes and saves all its references to points on the remote server with no namespacing. You can use the Git plumbing command `show-ref` to look at all your full reference names:

`git svn` 도구가 원격 브랜치의 이름을 어떻게 짓는가 알아두는 것이 중요하다. 일반적으로 Git 저장소를 복제할 경우 모든 브랜치는 `origin/[branch]` 처럼 원격 저장소의 이름을 가지고 모든 브랜치를 로컬에 복제해 놓는다. `git svn`은 우리가 여러 원격 저장소를 사용하지 않고 단 하나의 서버 저장소를 사용한다고 가정한다. 그렇기에 원격 저장소의 이름을 붙여서 원격 브랜치를 관리하지 않는다. Git의 좀 더 내부적인 명령인 `show-ref` 명령으로 완전한 리모트 브랜치들의 이름을 확인해볼 수 있다.

	$ git show-ref
	1cbd4904d9982f386d87f88fce1c24ad7c0f0471 refs/heads/master
	aee1ecc26318164f355a883f5d99cff0c852d3c4 refs/remotes/my-calc-branch
	03d09b0e2aad427e34a6d50ff147128e76c0e0f5 refs/remotes/tags/2.0.2
	50d02cc0adc9da4319eeba0900430ba219b9c376 refs/remotes/tags/release-2.0.1
	4caaa711a50c77879a91b8b90380060f672745cb refs/remotes/tags/release-2.0.2
	1c4cb508144c513ff1214c3488abe66dcb92916f refs/remotes/tags/release-2.0.2rc1
	1cbd4904d9982f386d87f88fce1c24ad7c0f0471 refs/remotes/trunk

A normal Git repository looks more like this:

일반적인 Git 저장소의 경우라면 다음과 비슷할 것이다:

	$ git show-ref
	83e38c7a0af325a9722f2fdc56b10188806d83a1 refs/heads/master
	3e15e38c198baac84223acfc6224bb8b99ff2281 refs/remotes/gitserver/master
	0a30dd3b0c795b80212ae723640d4e5d48cabdff refs/remotes/origin/master
	25812380387fdd55f916652be4881c6f11600d6f refs/remotes/origin/testing

You have two remote servers: one named `gitserver` with a `master` branch; and another named `origin` with two branches, `master` and `testing`. 

`master` 브랜치를 가진 `gitserver` 서버 저장소와 `master`, `testing` 두 브랜치를 가진 `origin` 이라는 서버 저장소를 확인해 볼 수 있다.

Notice how in the example of remote references imported from `git svn`, tags are added as remote branches, not as real Git tags. Your Subversion import looks like it has a remote named tags with branches under it.

`git svn`으로 가져온 저장소의 경우 태그가 일반적인 Git 태그가 아니라 원격 브랜치로 등록되는점 등을 잘 기억해두자. Subversion 태그는 tags라는 이름의 원격 서버의 브랜치처럼 보일 것이다.

### Committing Back to Subversion / Subversion 서버에 커밋하기 ###

Now that you have a working repository, you can do some work on the project and push your commits back upstream, using Git effectively as a SVN client. If you edit one of the files and commit it, you have a commit that exists in Git locally that doesn’t exist on the Subversion server:

자 이제 작업할 Git 저장소는 준비되었고, 무엇인가 수정을 하면 이제는 서버로 고친 내용을 Push 해야할 때가 왔다. Git을 Subversion의 클라이언트로 사용해서 효율적으로 수정한 내용을 전송할 수 있다. 어떤 파일을 수정하고 커밋을 하게 되면, 그 수정한 내용은 Git의 로컬 저장소에 저장되지만 Subversion 서버에는 아직 반영되지 않는다.

	$ git commit -am 'Adding git-svn instructions to the README'
	[master 97031e5] Adding git-svn instructions to the README
	 1 files changed, 1 insertions(+), 1 deletions(-)

Next, you need to push your change upstream. Notice how this changes the way you work with Subversion — you can do several commits offline and then push them all at once to the Subversion server. To push to a Subversion server, you run the `git svn dcommit` command:

이제 서버로 수정한 내용을 전송할 때가 왔다. 하나 유심히 살펴볼 부분은 Git 저장소에 여러개의 커밋을 쌓아놓고 Subversion 서버로는 해당 커밋을 한번에 보낼 수 있다는 점이다. 서버로 Push하기 위해서 `git svn dcommit` 명령을 사용한다:

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	       M      README.txt
	Committed r79
	       M      README.txt
	r79 = 938b1a547c2cc92033b74d32030e86468294a5c8 (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

This takes all the commits you’ve made on top of the Subversion server code, does a Subversion commit for each, and then rewrites your local Git commit to include a unique identifier. This is important because it means that all the SHA-1 checksums for your commits change. Partly for this reason, working with Git-based remote versions of your projects concurrently with a Subversion server isn’t a good idea. If you look at the last commit, you can see the new `git-svn-id` that was added:

이 명령은 앞서 가져온 Subversion 코드 이후에 추가된 커밋들을 각각 Subversion 커밋으로 만들고 다시 로컬 Git 커밋을 재작성 한다. 커밋이 다시 써지기 때문에 이미 저장되어 있던 커밋의 SHA-1 체크섬이 바뀌게 된다는 점을 알아둘 필요가 있다. 이런 이유에서 Git 기반의 리모트를 Subversion 서버와 함께 사용하는 것은 좋은 생각이 아니다. 업데이트된 커밋을 살펴보면 아래와 같이 `git-svn-id` 내용이 추가된 것을 볼 수 있다:

	$ git log -1
	commit 938b1a547c2cc92033b74d32030e86468294a5c8
	Author: schacon <schacon@4c93b258-373f-11de-be05-5f7a86268029>
	Date:   Sat May 2 22:06:44 2009 +0000

	    Adding git-svn instructions to the README

	    git-svn-id: file:///tmp/test-svn/trunk@79 4c93b258-373f-11de-be05-5f7a86268029

Notice that the SHA checksum that originally started with `97031e5` when you committed now begins with `938b1a5`. If you want to push to both a Git server and a Subversion server, you have to push (`dcommit`) to the Subversion server first, because that action changes your commit data.

원래 `97031e5`로 시작하는 SHA 체크섬이 지금은 `938b1a5`로 시작하는 체크섬으로 업데이트 되었다. 만약 Git 서버와 Subversion 서버에 함께 Push 하고 싶다면 우선 Subversion 서버에 `dcommit`으로 먼저 Push를 하고 그 다음에 업데이트된 커밋을 Git 서버에 Push 해야 한다.

### Pulling in New Changes / 새로운 변경사항 받아오기 ###

If you’re working with other developers, then at some point one of you will push, and then the other one will try to push a change that conflicts. That change will be rejected until you merge in their work. In `git svn`, it looks like this:

다른 개발자들과 함께 일하는 과정에서 다른 개발자가 Push한 상태에서 Push를 하게 되면 Conflict가 발생하게 될 경우도 있다. 이런 경우 변경사항을 해당 Conflict를 처리하지 않으면 서버로 Push하는 것은 거절된다. `git svn`을 사용하는 경우 아래와 같은 상황일 것이다:

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	Merge conflict during commit: Your file or directory 'README.txt' is probably \
	out-of-date: resource out of date; try updating at /Users/schacon/libexec/git-\
	core/git-svn line 482

To resolve this situation, you can run `git svn rebase`, which pulls down any changes on the server that you don’t have yet and rebases any work you have on top of what is on the server:

이런 상황에서 문제를 해결하기 위해 `git svn rebase` 명령을 사용할 수 있다. 이 명령은 서버에서 변경사항을 내려받고 그 다음에 로컬의 변경사항들을 그 위에 적용한다:

	$ git svn rebase
	       M      README.txt
	r80 = ff829ab914e8775c7c025d741beb3d523ee30bc4 (trunk)
	First, rewinding head to replay your work on top of it...
	Applying: first user change

Now, all your work is on top of what is on the Subversion server, so you can successfully `dcommit`:

그러고 나면 변경사항들이 서버의 코드 위에 적용이 되었기 때문에 성공적으로 `dcommit` 명령을 마칠 수 있다:

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	       M      README.txt
	Committed r81
	       M      README.txt
	r81 = 456cbe6337abe49154db70106d1836bc1332deed (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

It’s important to remember that unlike Git, which requires you to merge upstream work you don’t yet have locally before you can push, `git svn` makes you do that only if the changes conflict. If someone else pushes a change to one file and then you push a change to another file, your `dcommit` will work fine:

Push하기 전에 서버의 내용을 Merge하는 Git과 달리 `git svn`은 Conflict가 발생했을때에만 사용자에게 서버의 내용으로 업데이트하기를 요청한다는 점을 알아두는 것이 중요하다. 만약 서로 다른 파일을 수정했을 경우라면 `dcommit`은 성공적으로 수행되었을 것이다:

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	       M      configure.ac
	Committed r84
	       M      autogen.sh
	r83 = 8aa54a74d452f82eee10076ab2584c1fc424853b (trunk)
	       M      configure.ac
	r84 = cdbac939211ccb18aa744e581e46563af5d962d0 (trunk)
	W: d2f23b80f67aaaa1f6f5aaef48fce3263ac71a92 and refs/remotes/trunk differ, \
	  using rebase:
	:100755 100755 efa5a59965fbbb5b2b0a12890f1b351bb5493c18 \
	  015e4c98c482f0fa71e4d5434338014530b37fa6 M   autogen.sh
	First, rewinding head to replay your work on top of it...
	Nothing to do.

This is important to remember, because the outcome is a project state that didn’t exist on either of your computers when you pushed. If the changes are incompatible but don’t conflict, you may get issues that are difficult to diagnose. This is different than using a Git server — in Git, you can fully test the state on your client system before publishing it, whereas in SVN, you can’t ever be certain that the states immediately before commit and after commit are identical.

이 부분이 왜 중요하냐면 Push하고 난 프로젝트 코드의 상태가 Push하기 이전의 상태와 같지 않기 때문이다. 변경사항이 원하는 바 대로 적용되지 않았음에도 Conflict가 발생하지 않았다면 코드를 분석하기가 까다로워진다. 이러한 부분이 Git과 다른점인데 Git에서는 서버로 보내기 전에 프로젝트 코드의 모든 상태를 테스트해 볼 수 있다. 하지만 SVN에서는 서버로 커밋하기 이전의 상태와 그 이후의 상태가 동일한 상태라는 것을 확신할 수 없다.

You should also run this command to pull in changes from the Subversion server, even if you’re not ready to commit yourself. You can run `git svn fetch` to grab the new data, but `git svn rebase` does the fetch and then updates your local commits.

또는 Subversion 서버로부터 변경사항을 가져오기 위한 목적으로도 `git svn rebase` 명령을 사용할 수 있다. 커밋을 보낼 준비가 되지 않았다 해도 말이다. `git svn fetch` 명령을 사용할수도 있지만 `git svn rebase` 명령을 통해 변경사항을 가져오고 로컬에 적용까지 한 번에 할 수 있다.

	$ git svn rebase
	       M      generate_descriptor_proto.sh
	r82 = bd16df9173e424c6f52c337ab6efa7f7643282f1 (trunk)
	First, rewinding head to replay your work on top of it...
	Fast-forwarded master to refs/remotes/trunk.

Running `git svn rebase` every once in a while makes sure your code is always up to date. You need to be sure your working directory is clean when you run this, though. If you have local changes, you must either stash your work or temporarily commit it before running `git svn rebase` — otherwise, the command will stop if it sees that the rebase will result in a merge conflict.

수시로 `git svn rebase` 명령을 사용한다면 로컬 코드는 항상 서버의 새로운 변경사항이 적용되어 있을 것이다. 이 명령을 사용하기 전에 항상 작업하고 있는 디렉토리의 변경상태를 깨끗하게 유지하는 것이 좋다. 로컬 변경사항이 있는 경우 Stash를 하거나 임시로 커밋을 하고 난 후에 `git svn rebase` 명령을 실행하는 것이 좋다. 그렇지 않으면 Conflict가 발생했을 때 실행되던 명령이 중지되는 것을 보게 될 것이다.

### Git Branching Issues / Git 브랜치 문제 ###

When you’ve become comfortable with a Git workflow, you’ll likely create topic branches, do work on them, and then merge them in. If you’re pushing to a Subversion server via git svn, you may want to rebase your work onto a single branch each time instead of merging branches together. The reason to prefer rebasing is that Subversion has a linear history and doesn’t deal with merges like Git does, so git svn follows only the first parent when converting the snapshots into Subversion commits.

Git의 워크플로우에 익숙해졌다면 뭔가 일을 할 때 토픽 브랜치를 만들고 다시 Merge하는 방식을 쓰려고 할 것이다. `git svn`을 사용하여 Subversion 서버로 Push하려고 할 때 먼저 서버의 변경사항을 로컬에 적용해야 하는데 이 때 여러 브랜치를 Merge 하는 것이 아니라 하나의 브랜치에 서버의 내용을 Rebase로 적용하게 될 것이다. Git과 달리 Subversion의 경우 히스토리를 일직선으로 관리하기 때문에 Rebase를 사용하는 편이 더 나을 것이며 `git svn` 또한 커밋을 Subversion 커밋으로 변경할 때 하나의 부모 커밋만 가지도록 변경하기 때문이다.

Suppose your history looks like the following: you created an `experiment` branch, did two commits, and then merged them back into `master`. When you `dcommit`, you see output like this:

`experiment` 브랜치를 하나 만들고 2개의 변경사항을 커밋하고 `master` 브랜치로 Merge 했을 때 변경사항의 히스토리를 가지고 `dcommit` 명령을 수행하면 아래와 같은 모양이 될 것이다.

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	       M      CHANGES.txt
	Committed r85
	       M      CHANGES.txt
	r85 = 4bfebeec434d156c36f2bcd18f4e3d97dc3269a2 (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk
	COPYING.txt: locally modified
	INSTALL.txt: locally modified
	       M      COPYING.txt
	       M      INSTALL.txt
	Committed r86
	       M      INSTALL.txt
	       M      COPYING.txt
	r86 = 2647f6b86ccfcaad4ec58c520e369ec81f7c283c (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

Running `dcommit` on a branch with merged history works fine, except that when you look at your Git project history, it hasn’t rewritten either of the commits you made on the `experiment` branch — instead, all those changes appear in the SVN version of the single merge commit.

브랜치를 Merge한 히스토리(역주: Not FF)가 있는 브랜치에서 `dcommit` 명령을 수행하고 Git 히스토리를 살펴보면 `experiment` 브랜치의 커밋들이 재작성되지 않은 모습을 볼 수 있다. 대신 이 커밋들의 변경사항은 하나의 SVN 커밋으로 서버로 전송되어 기록되었다.

When someone else clones that work, all they see is the merge commit with all the work squashed into it; they don’t see the commit data about where it came from or when it was committed.

다른 사람이 이 변경사항을 내려받아 본다면 `experiment` 브랜치의 모든 변경사항이 하나의 Merge 커밋으로 모아져서 기록된 것으로 보인다. 어떤 브랜치인지 언제 쓰여졌는지는 알 수 없다.

### Subversion Branching / Subversion의 브랜치 기능 ###

Branching in Subversion isn’t the same as branching in Git; if you can avoid using it much, that’s probably best. However, you can create and commit to branches in Subversion using git svn.

Subversion의 브랜치 기능은 Git의 브랜치와 같지 않아서 가능한 사용을 하지 않는 것이 좋다. 하지만 불가능한 것은 아니며 `git svn`으로 Subversion 브랜치를 사용할 수 있다.

#### Creating a New SVN Branch ####

To create a new branch in Subversion, you run `git svn branch [branchname]`:

Subversion 브랜치를 새로 만들기 위해서 `git svn branch [branchname]` 명령을 사용한다:

	$ git svn branch opera
	Copying file:///tmp/test-svn/trunk at r87 to file:///tmp/test-svn/branches/opera...
	Found possible branch point: file:///tmp/test-svn/trunk => \
	  file:///tmp/test-svn/branches/opera, 87
	Found branch parent: (opera) 1f6bfe471083cbca06ac8d4176f7ad4de0d62e5f
	Following parent with do_switch
	Successfully followed parent
	r89 = 9b6fe0b90c5c9adf9165f700897518dbc54a7cbf (opera)

This does the equivalent of the `svn copy trunk branches/opera` command in Subversion and operates on the Subversion server. It’s important to note that it doesn’t check you out into that branch; if you commit at this point, that commit will go to `trunk` on the server, not `opera`.

위 명령은 Subversion의 `svn copy trunk branches/opera` 명령과 동일한 일을 한다. 여기서 주의해야할 점은 이 명령으로 브랜치를 옮겨가지는 않는다는 점이다. 지금 상황에서 커밋을 하면 `opera` 브랜치가 아니라 `trunk` 브랜치에 적용이 된다.

### Switching Active Branches / Subversion의 활성 브랜치 변경 ###

Git figures out what branch your dcommits go to by looking for the tip of any of your Subversion branches in your history — you should have only one, and it should be the last one with a `git-svn-id` in your current branch history.

`dcommit` 명령이 실행될 때 어떤 브랜치로 커밋들이 전송될지 Git은 히스토리를 뒤져서 Subversion 브랜치가 가장 최근에 가까운지 찾아본다. 최소한 하나 이상 커밋이 존재할텐데 커밋을 살펴보면 `git-svn-id`에 어떤 브랜치를 사용하고 있는지 확인할 수 있다.

If you want to work on more than one branch simultaneously, you can set up local branches to `dcommit` to specific Subversion branches by starting them at the imported Subversion commit for that branch. If you want an `opera` branch that you can work on separately, you can run

하나 이상의 여러 브랜치에서 동시에 작업을 하려고 한다면 현재 로컬 브랜치가 `dcommit` 명령이 실행되었을 때 어떤 Subversion 브랜치에 전송을 할 지 (by starting them at the imported Subversion commit for that branch) 지정해줄 수 있다. `opera` 브랜치를 따로 떼어내서 작업하려면 다음과 같이 지정한다:

	$ git branch opera remotes/opera

Now, if you want to merge your `opera` branch into `trunk` (your `master` branch), you can do so with a normal `git merge`. But you need to provide a descriptive commit message (via `-m`), or the merge will say "Merge branch opera" instead of something useful.

`opera` 브랜치를 `trunk` 브랜치(`master` 브랜치 역할)에 Merge하려면 일반적인 `git merge` 명령을 사용하면 된다. 하지만 `-m` 옵셥으로 적절한 커밋 메시지를 지정해주지 않으면 그저 "Mege branch opera"와 같은 메시지만 커밋에 남게 된다.

Remember that although you’re using `git merge` to do this operation, and the merge likely will be much easier than it would be in Subversion (because Git will automatically detect the appropriate merge base for you), this isn’t a normal Git merge commit. You have to push this data back to a Subversion server that can’t handle a commit that tracks more than one parent; so, after you push it up, it will look like a single commit that squashed in all the work of another branch under a single commit. After you merge one branch into another, you can’t easily go back and continue working on that branch, as you normally can in Git. The `dcommit` command that you run erases any information that says what branch was merged in, so subsequent merge-base calculations will be wrong — the dcommit makes your `git merge` result look like you ran `git merge --squash`. Unfortunately, there’s no good way to avoid this situation — Subversion can’t store this information, so you’ll always be crippled by its limitations while you’re using it as your server. To avoid issues, you should delete the local branch (in this case, `opera`) after you merge it into trunk.

`git merge` 명령을 사용했다고 해서 실제 Git의 Merge 결과가 되는 것은 아니다. 물론 Git의 Merge는 Subversion의 Merge보다 Merge Base(Merge의 기반이 되는 지점)을 쉽게 찾을 수 있기 때문에 쉽게 Merge를 할 수 있다. Subversion 서버에 Merge한 사항을 Push해야하는데 Subversion 서버는 기본적으로 이러한 하나 이상의 부모를 가지는 커밋을 처리할 수 있는 능력이 없다. 그렇기 때문에 `git svn`이 Subversion 서버로 Push하고 나면 Merge된 브랜치에서 변경된 사항은 단 하나의 커밋으로만 남겨놓는다. 이렇게 Merge된 커밋들은 쉽게 다시 이어서 사용하기가 어렵다. `dcommit` 명령을 수행하면 Merge된 브랜치의 정보를 어쩔 수 없이 잃어버리게 되는데 이로인해 Merge Base를 찾기도 힘들어진다. `dcommit` 명령을 실행하게 되면 `git merge`한 내용은 `git merge --squash` 한 것과 같은 결과가 된다. 불행히도 Subversion의 Branch의 Merge 정보를 처리할 수 없기 때문에 이렇게 어쩔 수 없는 상황을 피할 길이 없다. 이러한 문제를 되도록 피하기 위해서는 Merge한 후에 바로 해당 Branch(이 상황에서는 `opera`)를 삭제하도록 한다.

### Subversion Commands / Subversion 명령 ###

The `git svn` toolset provides a number of commands to help ease the transition to Git by providing some functionality that’s similar to what you had in Subversion. Here are a few commands that give you what Subversion used to.

`git svn` 명령은 Subversion의 일부 기능을 지원하는 비슷한 명령을 제공하여 Git으로 쉽게 이전하도록 돕는다. 아래 몇가지 명령은 Subversion에서 익히 사용하던 것들이다. 

#### SVN Style History / SVN 형식의 히스토리 ####

If you’re used to Subversion and want to see your history in SVN output style, you can run `git svn log` to view your commit history in SVN formatting:

Subversion에 익숙한 사람은 Git 히스토리를 SVN 형식의 히스토리로 보았으면 할지도 모른다. `git svn log` 명령으로 SVN 형식의 히스토리를 볼 수 있다:

	$ git svn log
	------------------------------------------------------------------------
	r87 | schacon | 2009-05-02 16:07:37 -0700 (Sat, 02 May 2009) | 2 lines

	autogen change

	------------------------------------------------------------------------
	r86 | schacon | 2009-05-02 16:00:21 -0700 (Sat, 02 May 2009) | 2 lines

	Merge branch 'experiment'

	------------------------------------------------------------------------
	r85 | schacon | 2009-05-02 16:00:09 -0700 (Sat, 02 May 2009) | 2 lines
	
	updated the changelog

You should know two important things about `git svn log`. First, it works offline, unlike the real `svn log` command, which asks the Subversion server for the data. Second, it only shows you commits that have been committed up to the Subversion server. Local Git commits that you haven’t dcommited don’t show up; neither do commits that people have made to the Subversion server in the meantime. It’s more like the last known state of the commits on the Subversion server.

`git svn log`에서 두 가지 정도 기억해 둘 점이 있다. 우선 오프라인에서 동작한다는 점인데 실제 `svn log` 명령어는 히스토리 데이터를 조회하기 위해 서버를 필요로 한다. 또한 이미 서버로 전송한 커밋만 출력해준다는 점이다. 아직 `dcommit` 명령으로 서버로 전송하지 않은 로컬 Git 커밋을 보여주지는 않는다. Subversion 서버에 기록되어 있지만 아직 내려받지 않은 변경사항도 보여주지 않는다. 즉, 현재 알고있는 Subversion 서버의 상태를 보여주는 것이다.

#### SVN Annotation / SVN 어노테이션 ####

Much as the `git svn log` command simulates the `svn log` command offline, you can get the equivalent of `svn annotate` by running `git svn blame [FILE]`. The output looks like this:

`git svn log` 명령이 `svn log` 명령을 오프라인에서 비슷하게 따라하는 것 처럼 `svn annotate` 명령 또한 `git svn blame [FILE]` 명령으로 실행 가능하다. 실행한 결과는 다음과 같을 것이다:

	$ git svn blame README.txt 
	 2   temporal Protocol Buffers - Google's data interchange format
	 2   temporal Copyright 2008 Google Inc.
	 2   temporal http://code.google.com/apis/protocolbuffers/
	 2   temporal 
	22   temporal C++ Installation - Unix
	22   temporal =======================
	 2   temporal 
	79    schacon Committing in git-svn.
	78    schacon 
	 2   temporal To build and install the C++ Protocol Buffer runtime and the Protocol
	 2   temporal Buffer compiler (protoc) execute the following:
	 2   temporal 

Again, it doesn’t show commits that you did locally in Git or that have been pushed to Subversion in the meantime.

다시한번 말하지만 이 명령 또한 아직 서버로 전송하지 않은 로컬 Git 저장소의 커밋을 출력하지 않는다.

#### SVN Server Information / SVN 서버 정보 ####

You can also get the same sort of information that `svn info` gives you by running `git svn info`:

`svn info` 명령은 `git svn info` 명령으로 대신할 수 있다:

	$ git svn info
	Path: .
	URL: https://schacon-test.googlecode.com/svn/trunk
	Repository Root: https://schacon-test.googlecode.com/svn
	Repository UUID: 4c93b258-373f-11de-be05-5f7a86268029
	Revision: 87
	Node Kind: directory
	Schedule: normal
	Last Changed Author: schacon
	Last Changed Rev: 87
	Last Changed Date: 2009-05-02 16:07:37 -0700 (Sat, 02 May 2009)

This is like `blame` and `log` in that it runs offline and is up to date only as of the last time you communicated with the Subversion server.

`blame`이나 `log`명령이 오프라인에서 동작 가능하듯 이 명령 또한 가장 마지막으로 서버에서 정보를 내려받은 시점에 대한 정보를 출력한다.

#### Ignoring What Subversion Ignores / Subversion에서 무시하는것 무시하기 ####

If you clone a Subversion repository that has `svn:ignore` properties set anywhere, you’ll likely want to set corresponding `.gitignore` files so you don’t accidentally commit files that you shouldn’t. `git svn` has two commands to help with this issue. The first is `git svn create-ignore`, which automatically creates corresponding `.gitignore` files for you so your next commit can include them.

실수로 커밋하지 말아야 할 파일을 커밋하지 않게 도와주는 `svn:ignore` 속성이 설정되어 있는 Subversion 저장소를 내려받으면 해당 위치에 `.gitignore` 파일 또한 만들고 싶은 생각이 들 것이다. `git svn`은 이런 소원을 들어주기 위해 두 가지 묘안을 갖고 잇다. 하나는 `git svn create-ignore` 명령이며 해당 위치에 다음 커밋에 포함시킬 수 있는 `.gitignore` 파일을 생성해준다.

The second command is `git svn show-ignore`, which prints to stdout the lines you need to put in a `.gitignore` file so you can redirect the output into your project exclude file:

두 번째 방법은 `git svn show-ignore` 명령이며 현재 화면에 `.gitignore`에 추가해야 할 목록을 보여준다. 리다이렉트를 사용하여 출력 결과를 프로젝트 exclude 파일에 보낼 수 있다:

	$ git svn show-ignore > .git/info/exclude

That way, you don’t litter the project with `.gitignore` files. This is a good option if you’re the only Git user on a Subversion team, and your teammates don’t want `.gitignore` files in the project.

이 방법을 사용하면 여러 `.gitignore` 파일들을 생성하지 않아도 된다. 여러분이 팀에서 Subversion 저장소를 사용하는 사람중에 Git을 쓰는 유일한 사람이라면, 팀원들이 `.gitignore` 파일을 보기 싫어한다면 이 방법이 좋은 묘안이 될 것이다.

### Git-Svn Summary / Git-Svn 요약 ###

The `git svn` tools are useful if you’re stuck with a Subversion server for now or are otherwise in a development environment that necessitates running a Subversion server. You should consider it crippled Git, however, or you’ll hit issues in translation that may confuse you and your collaborators. To stay out of trouble, try to follow these guidelines:

`git svn` 도구는 여러가지 이유로 Subversion 서버를 사용해야만 하는 상황에서 빛을 발한다. 하지만 Git의 모든 장점을 이용할 수는 없다. 또는 Git과 Subversion이 다른 이유로 혼란스러운 상황을 맞이할 수도 있다. 이런 문제로부터 거리를 두기 위해서는 아래의 가이드라인을 지키고자 노력할 필요가 있다:

* Keep a linear Git history that doesn’t contain merge commits made by `git merge`. Rebase any work you do outside of your mainline branch back onto it; don’t merge it in.
* Don’t set up and collaborate on a separate Git server. Possibly have one to speed up clones for new developers, but don’t push anything to it that doesn’t have a `git-svn-id` entry. You may even want to add a `pre-receive` hook that checks each commit message for a `git-svn-id` and rejects pushes that contain commits without it.

* Git 히스토리를 일직선으로 유지하라. `git merge`로 Merge 커밋이 생기지 않도록 하라. Merge를 하지 말고 Rebase를 사용하여 변경사항을 Master 브랜치에 적용하라.
* 따로 Git 저장소 서버를 두지 말라. 복제를 빨리 하기 위해서 하나쯤 두는 것은 무방하나 절대로 Git 서버에 Push하지는 말아야 한다. 혹은 `pre-receive` 훅 스크립트에서 커밋 메시지에 `git-svn-id` 내용을 포함하는 커밋은 거절하도록 설정해야 한다.

If you follow those guidelines, working with a Subversion server can be more bearable. However, if it’s possible to move to a real Git server, doing so can gain your team a lot more.

이러한 가이드라인을 지키면 Subversion 서버를 사용하는 것도 쓸만하다. 그렇다고 하더라도 진짜 Git 서버를 사용할 수 있는 상황이라면 진짜 Git 서버를 사용하는 것이 훨씬 얻을 것이 많다.

## Migrating to Git ##

If you have an existing codebase in another VCS but you’ve decided to start using Git, you must migrate your project one way or another. This section goes over some importers that are included with Git for common systems and then demonstrates how to develop your own custom importer.

다른 버전관리 시스템으로 관리하는 프로젝트를 Git 기반으로 사용하고 싶다면 우선 프로젝트를 Git 프로젝트로 이전(Migrate)해야 한다. 이번 절에서는 Git에 포함된 가져오기 도구들을 살펴보고 널리 쓰이는 버전관리 도구로부터 가져오는 방법이나 새로 가져오기 도구를 만들어서 사용하는 방법을 알아볼 것이다.

### Importing / 가져오기 ###

You’ll learn how to import data from two of the bigger professionally used SCM systems — Subversion and Perforce — both because they make up the majority of users I hear of who are currently switching, and because high-quality tools for both systems are distributed with Git.

널리 사용되는 버전 관리 시스템 중 Subversion과 Perforce로부터 프로젝트를 이전하는 방법을 살펴볼 것이다. 이 두가지 버전 관리 시스템은 들어봤던 이야기 중 Git 기반으로 프로젝트를 변경하고자 하는 주요 시스템이었으며 Git에도 관련 시스템의 가져오기 도구가 포함되어 배포된다.

### Subversion ###

If you read the previous section about using `git svn`, you can easily use those instructions to `git svn clone` a repository; then, stop using the Subversion server, push to a new Git server, and start using that. If you want the history, you can accomplish that as quickly as you can pull the data out of the Subversion server (which may take a while).

앞서 살펴본 `git svn` 절을 읽어봤다면 `git svn clone` 명령으로 손쉽게 저장소를 가져올 수 있다. 가져오고 난 다음에 Subversion 서버를 중지하고 새로 Git 서버를 사용하면 된다. 히스토리 정보 또한 (느린) Subversion 서버 없이 로컬에서 조회해 볼 수 있다.

However, the import isn’t perfect; and because it will take so long, you may as well do it right. The first problem is the author information. In Subversion, each person committing has a user on the system who is recorded in the commit information. The examples in the previous section show `schacon` in some places, such as the `blame` output and the `git svn log`. If you want to map this to better Git author data, you need a mapping from the Subversion users to the Git authors. Create a file called `users.txt` that has this mapping in a format like this:

이 가져오기 기능이 완벽한 것은 아니며 가져오기 과정 또한 시간과 수고가 어느정도 들기 때문에 바로 뭔가 작업을 시작 할 수 있는 것은 아니다. 우선 저자 정보에 문제가 있다. Subversion의 각 커밋에 커밋을 작성한 사람의 정보가 기록되어 있는데 예를 들어 앞 절에서 `blame`이나 `git svn log`와 같은 명령에서 `schacon`이라는 이름을 볼 수 있다. 이 저자 정보를 좀 더 나은 Git 저자 정보로 변경하기 위해서 Subversion 사용자 이름과 Git 저자 간에 연결을 해 주어야 한다. `users.txt`라는 파일을 만들어서 다음과 같이 연결 규칙을 만든다.

	schacon = Scott Chacon <schacon@geemail.com>
	selse = Someo Nelse <selse@geemail.com>

To get a list of the author names that SVN uses, you can run this:

SVN에 기록된 저자 이름 목록을 다음 명령으로 조회해 볼 수 있다.

	$ svn log --xml | grep author | sort -u | perl -pe 's/.>(.?)<./$1 = /'

That gives you the log output in XML format — you can look for the authors, create a unique list, and then strip out the XML. (Obviously this only works on a machine with `grep`, `sort`, and `perl` installed.) Then, redirect that output into your users.txt file so you can add the equivalent Git user data next to each entry.

우선 앞에서부터 살펴보면 XML 형식의 SVN 로그를 출력하고, 그 안에서 author 정보를 찾아 중복된 것을 제거하고 XML 태그를 떼어버린다(물론 `grep`, `sort`, `perl` 명령이 동작하는 시스템에서 작동할 것이다). 이 결과를 가지고 `users.txt` 만들어 저자 정보를 연결한다.

You can provide this file to `git svn` to help it map the author data more accurately. You can also tell `git svn` not to include the metadata that Subversion normally imports, by passing `--no-metadata` to the `clone` or `init` command. This makes your `import` command look like this:

이렇게 만들어진 파일을 `git svn` 명령에 전달하면 보다 의미있는 저자 정보를 Git 저장소에 기록할 수 있다. 또한 `git svn`의 `clone`이나 `init` 명령에 `--no-metadata` 옵션을 사용하며 Subversion에서 일반적으로 가져오게 되는 메타데이터를 저장하지 않도록 설정할 수 있다. 해당 명령은 아래와 같은 모습일 것이다:

	$ git-svn clone http://my-project.googlecode.com/svn/ \
	      --authors-file=users.txt --no-metadata -s my_project

Now you should have a nicer Subversion import in your `my_project` directory. Instead of commits that look like this

`my_project` 디렉토리에 좀 더 나은 Subversion으로부터 가져오기 작업을 완료하였다. 바로 아래와 같은 모습이 아니라:

	commit 37efa680e8473b615de980fa935944215428a35a
	Author: schacon <schacon@4c93b258-373f-11de-be05-5f7a86268029>
	Date:   Sun May 3 00:12:22 2009 +0000

	    fixed install - go to trunk

	    git-svn-id: https://my-project.googlecode.com/svn/trunk@94 4c93b258-373f-11de-
	    be05-5f7a86268029

they look like this:

이와 같은 모습이 될 것이다:

	commit 03a8785f44c8ea5cdb0e8834b7c8e6c469be2ff2
	Author: Scott Chacon <schacon@geemail.com>
	Date:   Sun May 3 00:12:22 2009 +0000

	    fixed install - go to trunk

Not only does the Author field look a lot better, but the `git-svn-id` is no longer there, either.

저자 정보 항목이 훨씬 나아졌고 `git-svn-id` 항목 또한 기록되지 않았다.

You need to do a bit of `post-import` cleanup. For one thing, you should clean up the weird references that `git svn` set up. First you’ll move the tags so they’re actual tags rather than strange remote branches, and then you’ll move the rest of the branches so they’re local.

이제 약간의 `post-import` 정리 작업을 해야 한다. `git svn`이 설정해놓은 이상한 모양의 브랜치나 Tag들을 제거해 주어야 한다. 우선 태그가 Git에서도 사용해야 하는 의미있는 태그라면 원격 브랜치로 설정되어 있는 SVN 태그들을 실제 Git 태그로 옮겨와야 한다.

To move the tags to be proper Git tags, run

Git에 알맞는 태그로 옮기려면 다음과 같이 한다

	$ cp -Rf .git/refs/remotes/tags/* .git/refs/tags/
	$ rm -Rf .git/refs/remotes/tags

This takes the references that were remote branches that started with `tag/` and makes them real (lightweight) tags.

위 명령은 `tag/` 로 시작하는 원격 브랜치들을 실제 Git의 (Lightweight) Tag로 옮겨준다.

Next, move the rest of the references under `refs/remotes` to be local branches:

또한 원격 브랜치로 설정되어 있는 브랜치를 로컬로 옮긴다:

	$ cp -Rf .git/refs/remotes/* .git/refs/heads/
	$ rm -Rf .git/refs/remotes

Now all the old branches are real Git branches and all the old tags are real Git tags. The last thing to do is add your new Git server as a remote and push to it. Here is an example of adding your server as a remote:

이렇게 하고 나면 이전의 브랜치나 태그는 완전한 Git 브랜치나 태그로 되었을 것이다. 이제 마지막으로 남은 작업은 새 Git 서버를 리모트로 추가를 하고 지금까지의 작업을  Push 하는 것이다. 리모트로 새 서버를 추가하려면 아래와 같이 한다:

	$ git remote add origin git@my-git-server:myrepository.git

Because you want all your branches and tags to go up, you can now run this:

모든 브랜치와 태그를 다 Push하기 위해서 아래와 같은 옵션으로 Push 한다:

	$ git push origin --all

All your branches and tags should be on your new Git server in a nice, clean import.

이제 깔끔하게 가져오기 완료된 저장소와 모든 브랜치와 태그가 서버에도 똑같이 존재할 것이다.

### Perforce ###

The next system you’ll look at importing from is Perforce. A Perforce importer is also distributed with Git, but only in the `contrib` section of the source code — it isn’t available by default like `git svn`. To run it, you must get the Git source code, which you can download from git.kernel.org:

다음으로 살펴볼 가져올 저장소 시스템은 Perforce이다. Preforce로부터 가져오는 도구는 Git과 같이 배포되지만 소스코드의 `contrib` 부분에만 있기 때문에 `git svn` 처럼 기본적으로 Git에 내장되어 배포되지는 않는다. Perforce 가져오기를 사용하려면 우선 Git 소스코드를 git.kernel.org 사이트로부터 다운로드 받아야 한다.

	$ git clone git://git.kernel.org/pub/scm/git/git.git
	$ cd git/contrib/fast-import

In this `fast-import` directory, you should find an executable Python script named `git-p4`. You must have Python and the `p4` tool installed on your machine for this import to work. For example, you’ll import the Jam project from the Perforce Public Depot. To set up your client, you must export the P4PORT environment variable to point to the Perforce depot:

이 `fast-import` 디렉토리에 `git-p4` 라는 파이썬 스크립트가 있어야 한다. 또한 파이썬과 `p4`가 설치되어 있어야 가져오기 스크립트가 동작한다. Jam 프로젝트를 Perforce Public Depot에서 가져오는 예제를 살펴보자. 우선 P4PORT 환경변수를 Perfoce Depot의 주소로 설정한다.

	$ export P4PORT=public.perforce.com:1666

Run the `git-p4 clone` command to import the Jam project from the Perforce server, supplying the depot and project path and the path into which you want to import the project:

`git-p4 clone` 명령에 depot과 프로젝트 경로 그리고 프로젝트를 가져올 경로를 적어서 실행하면 Perforce 서버에서 Jam 프로젝트 가져온다.

	$ git-p4 clone //public/jam/src@all /opt/p4import
	Importing from //public/jam/src@all into /opt/p4import
	Reinitialized existing Git repository in /opt/p4import/.git/
	Import destination: refs/remotes/p4/master
	Importing revision 4409 (100%)

If you go to the `/opt/p4import` directory and run `git log`, you can see your imported work:

`/opt/p4import` 디렉토리로 이동하여 `git log` 명령을 실행해 보면 가져오기 된 프로젝트의 정보를 볼 수 있다:

	$ git log -2
	commit 1fd4ec126171790efd2db83548b85b1bbbc07dc2
	Author: Perforce staff <support@perforce.com>
	Date:   Thu Aug 19 10:18:45 2004 -0800

	    Drop 'rc3' moniker of jam-2.5.  Folded rc2 and rc3 RELNOTES into
	    the main part of the document.  Built new tar/zip balls.

	    Only 16 months later.

	    [git-p4: depot-paths = "//public/jam/src/": change = 4409]

	commit ca8870db541a23ed867f38847eda65bf4363371d
	Author: Richard Geiger <rmg@perforce.com>
	Date:   Tue Apr 22 20:51:34 2003 -0800

	    Update derived jamgram.c

	    [git-p4: depot-paths = "//public/jam/src/": change = 3108]

You can see the `git-p4` identifier in each commit. It’s fine to keep that identifier there, in case you need to reference the Perforce change number later. However, if you’d like to remove the identifier, now is the time to do so — before you start doing work on the new repository. You can use `git filter-branch` to remove the identifier strings en masse:

각 커밋에서 `git-p4` ID을 찾아볼 수 있다. 나중에 Perforce Change Number를 찾아보기 위해서 ID를 커밋에 저장한 채로 유지하는 것이 좋다. 하지만 ID를 지우고자 한다면 딱 지금이 그 작업을 할 수 있는 시점이다. `git filter-branch` 명령으로 한방에(en masse) 제거할 수 있다.

	$ git filter-branch --msg-filter '
	        sed -e "/^\[git-p4:/d"
	'
	Rewrite 1fd4ec126171790efd2db83548b85b1bbbc07dc2 (123/123)
	Ref 'refs/heads/master' was rewritten

If you run `git log`, you can see that all the SHA-1 checksums for the commits have changed, but the `git-p4` strings are no longer in the commit messages:

`git log` 명령을 실행해보면 모든 SHA-1 체크섬이 변경된 것을 확인할 수 있으며 커밋 메시지에서 `git-p4` ID 문자도 제거된 것을 확인할 수 있다.

	$ git log -2
	commit 10a16d60cffca14d454a15c6164378f4082bc5b0
	Author: Perforce staff <support@perforce.com>
	Date:   Thu Aug 19 10:18:45 2004 -0800

	    Drop 'rc3' moniker of jam-2.5.  Folded rc2 and rc3 RELNOTES into
	    the main part of the document.  Built new tar/zip balls.

	    Only 16 months later.

	commit 2b6c6db311dd76c34c66ec1c40a49405e6b527b2
	Author: Richard Geiger <rmg@perforce.com>
	Date:   Tue Apr 22 20:51:34 2003 -0800

	    Update derived jamgram.c

Your import is ready to push up to your new Git server.

이제 가져온 저장소를 새 Git 서버에 올릴 준비가 되었다.

### A Custom Importer / 사용자 정의 가져오기 ###

If your system isn’t Subversion or Perforce, you should look for an importer online — quality importers are available for CVS, Clear Case, Visual Source Safe, even a directory of archives. If none of these tools works for you, you have a rarer tool, or you otherwise need a more custom importing process, you should use `git fast-import`. This command reads simple instructions from stdin to write specific Git data. It’s much easier to create Git objects this way than to run the raw Git commands or try to write the raw objects (see Chapter 9 for more information). This way, you can write an import script that reads the necessary information out of the system you’re importing from and prints straightforward instructions to stdout. You can then run this program and pipe its output through `git fast-import`.

To quickly demonstrate, you’ll write a simple importer. Suppose you work in current, you back up your project by occasionally copying the directory into a time-stamped `back_YYYY_MM_DD` backup directory, and you want to import this into Git. Your directory structure looks like this:

	$ ls /opt/import_from
	back_2009_01_02
	back_2009_01_04
	back_2009_01_14
	back_2009_02_03
	current

In order to import a Git directory, you need to review how Git stores its data. As you may remember, Git is fundamentally a linked list of commit objects that point to a snapshot of content. All you have to do is tell `fast-import` what the content snapshots are, what commit data points to them, and the order they go in. Your strategy will be to go through the snapshots one at a time and create commits with the contents of each directory, linking each commit back to the previous one.

As you did in the "An Example Git Enforced Policy" section of Chapter 7, we’ll write this in Ruby, because it’s what I generally work with and it tends to be easy to read. You can write this example pretty easily in anything you’re familiar with — it just needs to print the appropriate information to stdout. And, if you are running on Windows, this means you'll need to take special care to not introduce carriage returns at the end your lines — git fast-import is very particular about just wanting line feeds (LF) not the carriage return line feeds (CRLF) that Windows uses.

To begin, you’ll change into the target directory and identify every subdirectory, each of which is a snapshot that you want to import as a commit. You’ll change into each subdirectory and print the commands necessary to export it. Your basic main loop looks like this:

	last_mark = nil

	# loop through the directories
	Dir.chdir(ARGV[0]) do
	  Dir.glob("*").each do |dir|
	    next if File.file?(dir)

	    # move into the target directory
	    Dir.chdir(dir) do 
	      last_mark = print_export(dir, last_mark)
	    end
	  end
	end

You run `print_export` inside each directory, which takes the manifest and mark of the previous snapshot and returns the manifest and mark of this one; that way, you can link them properly. "Mark" is the `fast-import` term for an identifier you give to a commit; as you create commits, you give each one a mark that you can use to link to it from other commits. So, the first thing to do in your `print_export` method is generate a mark from the directory name:

	mark = convert_dir_to_mark(dir)

You’ll do this by creating an array of directories and using the index value as the mark, because a mark must be an integer. Your method looks like this:

	$marks = []
	def convert_dir_to_mark(dir)
	  if !$marks.include?(dir)
	    $marks << dir
	  end
	  ($marks.index(dir) + 1).to_s
	end

Now that you have an integer representation of your commit, you need a date for the commit metadata. Because the date is expressed in the name of the directory, you’ll parse it out. The next line in your `print_export` file is

	date = convert_dir_to_date(dir)

where `convert_dir_to_date` is defined as

	def convert_dir_to_date(dir)
	  if dir == 'current'
	    return Time.now().to_i
	  else
	    dir = dir.gsub('back_', '')
	    (year, month, day) = dir.split('_')
	    return Time.local(year, month, day).to_i
	  end
	end

That returns an integer value for the date of each directory. The last piece of meta-information you need for each commit is the committer data, which you hardcode in a global variable:

	$author = 'Scott Chacon <schacon@example.com>'

Now you’re ready to begin printing out the commit data for your importer. The initial information states that you’re defining a commit object and what branch it’s on, followed by the mark you’ve generated, the committer information and commit message, and then the previous commit, if any. The code looks like this:

	# print the import information
	puts 'commit refs/heads/master'
	puts 'mark :' + mark
	puts "committer #{$author} #{date} -0700"
	export_data('imported from ' + dir)
	puts 'from :' + last_mark if last_mark

You hardcode the time zone (-0700) because doing so is easy. If you’re importing from another system, you must specify the time zone as an offset. 
The commit message must be expressed in a special format:

	data (size)\n(contents)

The format consists of the word data, the size of the data to be read, a newline, and finally the data. Because you need to use the same format to specify the file contents later, you create a helper method, `export_data`:

	def export_data(string)
	  print "data #{string.size}\n#{string}"
	end

All that’s left is to specify the file contents for each snapshot. This is easy, because you have each one in a directory — you can print out the `deleteall` command followed by the contents of each file in the directory. Git will then record each snapshot appropriately:

	puts 'deleteall'
	Dir.glob("**/*").each do |file|
	  next if !File.file?(file)
	  inline_data(file)
	end

Note:	Because many systems think of their revisions as changes from one commit to another, fast-import can also take commands with each commit to specify which files have been added, removed, or modified and what the new contents are. You could calculate the differences between snapshots and provide only this data, but doing so is more complex — you may as well give Git all the data and let it figure it out. If this is better suited to your data, check the `fast-import` man page for details about how to provide your data in this manner.

The format for listing the new file contents or specifying a modified file with the new contents is as follows:

	M 644 inline path/to/file
	data (size)
	(file contents)

Here, 644 is the mode (if you have executable files, you need to detect and specify 755 instead), and inline says you’ll list the contents immediately after this line. Your `inline_data` method looks like this:

	def inline_data(file, code = 'M', mode = '644')
	  content = File.read(file)
	  puts "#{code} #{mode} inline #{file}"
	  export_data(content)
	end

You reuse the `export_data` method you defined earlier, because it’s the same as the way you specified your commit message data. 

The last thing you need to do is to return the current mark so it can be passed to the next iteration:

	return mark

NOTE: If you are running on Windows you'll need to make sure that you add one extra step. As metioned before, Windows uses CRLF for new line characters while git fast-import expects only LF. To get around this problem and make git fast-import happy, you need to tell ruby to use LF instead of CRLF:

	$stdout.binmode

That’s it. If you run this script, you’ll get content that looks something like this:

	$ ruby import.rb /opt/import_from 
	commit refs/heads/master
	mark :1
	committer Scott Chacon <schacon@geemail.com> 1230883200 -0700
	data 29
	imported from back_2009_01_02deleteall
	M 644 inline file.rb
	data 12
	version two
	commit refs/heads/master
	mark :2
	committer Scott Chacon <schacon@geemail.com> 1231056000 -0700
	data 29
	imported from back_2009_01_04from :1
	deleteall
	M 644 inline file.rb
	data 14
	version three
	M 644 inline new.rb
	data 16
	new version one
	(...)

To run the importer, pipe this output through `git fast-import` while in the Git directory you want to import into. You can create a new directory and then run `git init` in it for a starting point, and then run your script:

	$ git init
	Initialized empty Git repository in /opt/import_to/.git/
	$ ruby import.rb /opt/import_from | git fast-import
	git-fast-import statistics:
	---------------------------------------------------------------------
	Alloc'd objects:       5000
	Total objects:           18 (         1 duplicates                  )
	      blobs  :            7 (         1 duplicates          0 deltas)
	      trees  :            6 (         0 duplicates          1 deltas)
	      commits:            5 (         0 duplicates          0 deltas)
	      tags   :            0 (         0 duplicates          0 deltas)
	Total branches:           1 (         1 loads     )
	      marks:           1024 (         5 unique    )
	      atoms:              3
	Memory total:          2255 KiB
	       pools:          2098 KiB
	     objects:           156 KiB
	---------------------------------------------------------------------
	pack_report: getpagesize()            =       4096
	pack_report: core.packedGitWindowSize =   33554432
	pack_report: core.packedGitLimit      =  268435456
	pack_report: pack_used_ctr            =          9
	pack_report: pack_mmap_calls          =          5
	pack_report: pack_open_windows        =          1 /          1
	pack_report: pack_mapped              =       1356 /       1356
	---------------------------------------------------------------------

As you can see, when it completes successfully, it gives you a bunch of statistics about what it accomplished. In this case, you imported 18 objects total for 5 commits into 1 branch. Now, you can run `git log` to see your new history:

	$ git log -2
	commit 10bfe7d22ce15ee25b60a824c8982157ca593d41
	Author: Scott Chacon <schacon@example.com>
	Date:   Sun May 3 12:57:39 2009 -0700

	    imported from current

	commit 7e519590de754d079dd73b44d695a42c9d2df452
	Author: Scott Chacon <schacon@example.com>
	Date:   Tue Feb 3 01:00:00 2009 -0700

	    imported from back_2009_02_03

There you go — a nice, clean Git repository. It’s important to note that nothing is checked out — you don’t have any files in your working directory at first. To get them, you must reset your branch to where `master` is now:

	$ ls
	$ git reset --hard master
	HEAD is now at 10bfe7d imported from current
	$ ls
	file.rb  lib

You can do a lot more with the `fast-import` tool — handle different modes, binary data, multiple branches and merging, tags, progress indicators, and more. A number of examples of more complex scenarios are available in the `contrib/fast-import` directory of the Git source code; one of the better ones is the `git-p4` script I just covered.

## Summary / 요약 ##

You should feel comfortable using Git with Subversion or importing nearly any existing repository into a new Git one without losing data. The next chapter will cover the raw internals of Git so you can craft every single byte, if need be.
