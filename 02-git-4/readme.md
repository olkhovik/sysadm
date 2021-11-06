# **Домашняя работа к занятию «2.4. Инструменты Git.»**
## _Задача №1_
Найдите полный хеш и комментарий коммита, хеш которого начинается на `aefe`.

**`git show aefea --pretty=format:"Полный хеш коммита aefe: %H%nКомментарий коммита aefe: %s" -s`**
```
Полный хеш коммита aefe: aefead2207ef7e2aa5dc81a34aedf0cad4c32545
Комментарий коммита aefe: Update CHANGELOG.md
```
## _Задача №2_
Какому тегу соответствует коммит `85024d3`?

**`git show 85024d3 --pretty=format:"Тег коммита 85024d3: %D" -s`**
```
Тег коммита 85024d3: tag: v0.12.23
```
## _Задача №3_
Сколько родителей у коммита `b8d720`? Напишите их хеши.

**`git show b8d720 --pretty=format:"Родители коммита b8d720: %P"`**
```
Родители коммита b8d720: 56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b
```

или можно так: **`git show b8d720^@ -s`**
```
commit 56cd7859e05c36c06b56d013b55a252d0bb7e158
Merge: 58dcac4b7 ffbcf5581
Author: Chris Griggs <cgriggs@hashicorp.com>
Date:   Mon Jan 13 13:19:09 2020 -0800

    Merge pull request #23857 from hashicorp/cgriggs01-stable

    [cherry-pick]add checkpoint links

commit 9ea88f22fc6269854151c571162c5bcf958bee2b
Author: Chris Griggs <cgriggs@hashicorp.com>
Date:   Tue Jan 21 17:08:06 2020 -0800

    add/update community provider listings
```

## _Задача №4_
Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.

**`git log v0.12.23..v0.12.24 --pretty=format:"Хеш: %H%nКомментарий: %s"`**
```
Хеш: 33ff1c03bb960b332be3af2e333462dde88b279e
Комментарий: v0.12.24
Хеш: b14b74c4939dcab573326f4e3ee2a62e23e12f89
Комментарий: [Website] vmc provider links
Хеш: 3f235065b9347a758efadc92295b540ee0a5e26e
Комментарий: Update CHANGELOG.md
Хеш: 6ae64e247b332925b872447e9ce869657281c2bf
Комментарий: registry: Fix panic when server is unreachable
Хеш: 5c619ca1baf2e21a155fcdb4c264cc9e24a2a353
Комментарий: website: Remove links to the getting started guide's old location
Хеш: 06275647e2b53d97d4f0a19a0fec11f6d69820b5
Комментарий: Update CHANGELOG.md
Хеш: d5f9411f5108260320064349b757f55c09bc4b80
Комментарий: command: Fix bug when using terraform login on Windows
Хеш: 4b6d06cc5dcb78af637bbb19c198faff37a066ed
Комментарий: Update CHANGELOG.md
Хеш: dd01a35078f040ca984cdd349f18d0b67e486c35
Комментарий: Update CHANGELOG.md
Хеш: 225466bc3e5f35baa5d07197bbc079345b77525e
Комментарий: Cleanup after v0.12.23 release
```
или можно так: **`git show v0.12.23..v0.12.24 --pretty=format:"Хеш: %H%nКомментарий: %s" -s`**

## _Задача №5_
Найдите коммит в котором была создана функция `func providerSource`, её определение в коде выглядит так: `func providerSource(...)` (вместо троеточия перечислены аргументы).

**`git log -S "func providerSource" --oneline --reverse`**
```
8c928e835 main: Consult local directories as potential mirrors of providers
5af1e6234 main: Honor explicit provider_installation CLI config when present
```
Функция `func providerSource` была создана в коммите 8c928e83589d90a031f811fae52a81be7153e82f.

## _Задача №6_
Найдите все коммиты в которых была изменена функция `globalPluginDirs`.

1. Найдём файлы, где присутствует функция `globalPluginDirs`:

**`git grep -p globalPluginDirs`**
```
commands.go=func initCommands(
commands.go:            GlobalPluginDirs: globalPluginDirs(),
commands.go=func credentialsSource(config *cliconfig.Config) (auth.CredentialsSource, error) {
commands.go:    helperPlugins := pluginDiscovery.FindPlugins("credentials", globalPluginDirs())
internal/command/cliconfig/config_unix.go=func homeDir() (string, error) {
internal/command/cliconfig/config_unix.go:              // FIXME: homeDir gets called from globalPluginDirs during init, before
plugins.go=import (
plugins.go:// globalPluginDirs returns directories that should be searched for
plugins.go:func globalPluginDirs() []string {
```
**`git grep -c "func globalPluginDirs"`**
```
plugins.go:1
```
2. Найдём коммиты, в которых происходили изменения с функцией `globalPluginDirs` в файле `plugins.go`:

**`git log -L :globalPluginDirs:plugins.go -s --oneline`**
```
78b122055 Remove config.go and update things using its aliases
52dbf9483 keep .terraform.d/plugins for discovery
41ab0aef7 Add missing OS_ARCH dir to global plugin paths
66ebff90c move some more plugin search path logic to command
8364383c3 Push plugin discovery down into command package
```

## _Задача №7_
Кто автор функции `synchronizedWriters`?

1. Найдём первый коммит, где упоминается функция `synchronizedWriters`:

**`git log -S synchronizedWriters --oneline --reverse`**
```
5ac311e2a main: synchronize writes to VT100-faker on Windows
fd4f7eb0b remove prefixed io
bdfea50cc remove unused
```
2. Смотрим коммит 5ac311e2a, и видим в нём добавление функции `synchronizedWriters`:

**`git show 5ac311e2a`**

Автор функции `synchronizedWriters` Martin Atkins.
```
commit 5ac311e2a91e381e2f52234668b49ba670aa0fe5
Author: Martin Atkins <mart@degeneration.co.uk>
Date:   Wed May 3 16:25:41 2017 -0700
```

## _Подпись коммитов_
Настроил работу с GnuPG, ключами, в локальном репозитории и на Github.