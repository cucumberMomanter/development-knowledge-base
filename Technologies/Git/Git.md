### Cherry-pick
> команда, которая переносит коммит(ы) из одной ветки в другую.

#### Синтаксис

```shell
git cherry-pick <commit_hash>
```

```shell
git cherry-pick <start_commit_hash>^..<end_commit_hash>
```

- Здесь `^` используется для исключения стартового коммита.

#### Examples
1. Получаем `commit_hash` коммита, который нужно перенести из одной ветки в другую
Данной командой получаем всю информацию о коммитах:
```shell
git log --all --graph

или

git log
```
```shell
* commit 0894e4e362dd80acc6af0302a751151ef1d0fe59 '<- хэш коммита'
| Author: Юрий <******>
| Date:   Wed Jul 17 13:05:28 2024 +0000
| 
|     INFRA-1111 Change version socket IO
| 
* commit a0c4a63a590cb9523cd06a1b93605fadaf600f4d '<- хэш коммита'
| Author: Jenkins <******>
| Date:   Thu Jul 18 08:50:09 2024 +0000
| 
|     Update of patch version
| 

```

2. Переходим на ветку, в которую нужно перенести коммит
```shell
git checkout target-branch
```
3. Выполняем `cherry-pick`
```shell
git cherry-pick 0894e4e362dd80acc6af0302a751151ef1d0fe59
```