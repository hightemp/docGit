# Настройка Git - атрибуты Git

## Атрибуты Git

Некоторые из этих настроек также могут быть указаны для пути, так что Git применяет эти настройки только для подкаталога или подмножества файлов. Эти специфичные для пути настройки называются атрибутами Git и задаются либо в файле `.gitattributes` в одном из ваших каталогов (обычно в корне вашего проекта), либо в файле `.git/info/attribute`, если вы не хотите, чтобы файл атрибутов был зафиксирован в вашем проекте.

Используя атрибуты, вы можете делать такие вещи, как указание отдельных стратегий слияния для отдельных файлов или каталогов в вашем проекте, указывать Git, как создавать нетекстовые файлы, или использовать Git-фильтр для содержимого перед тем, как проверять его в Git или из него. В этом разделе вы узнаете о некоторых атрибутах, которые вы можете установить в своих путях в своем проекте Git, и увидите несколько примеров использования этой функции на практике.

### Двоичные файлы

Один интересный трюк, для которого вы можете использовать атрибуты Git - это сообщить Git, какие файлы являются двоичными (в противном случае он не сможет это выяснить), и дать Git специальные инструкции о том, как обрабатывать эти файлы. Например, некоторые текстовые файлы могут быть сгенерированы машиной и не могут быть различимы, тогда как некоторые двоичные файлы могут быть различимы. Вы увидите, как сказать Git, что есть что.

#### Идентификация бинарных файлов

Некоторые файлы выглядят как текстовые файлы, но в целом они должны рассматриваться как двоичные данные. Например, проекты Xcode в macOS содержат файл, который заканчивается на `.pbxproj`, который в основном представляет собой набор данных JSON (текстовый формат данных JavaScript), записанный на диск IDE, который записывает ваши параметры сборки и так далее. Хотя технически это текстовый файл (потому что это все UTF-8), вы не хотите рассматривать его как таковой, потому что это действительно легкая база данных - вы не можете объединить содержимое, если его изменяют два человека, и различия, как правило, не помогают. Файл предназначен для использования машиной. По сути, вы хотите рассматривать его как двоичный файл.

Чтобы заставить Git обрабатывать все файлы `pbxproj` как двоичные данные, добавьте следующую строку в ваш файл `.gitattributes`:

```ini
*.pbxproj binary
```

Теперь Git не будет пытаться конвертировать или исправлять ошибки CRLF; он также не будет пытаться вычислить или распечатать diff для изменений в этом файле, когда вы запустите `git show` или `git diff` в вашем проекте.

#### Сравнения Бинарных Файлов

Вы также можете использовать функциональность атрибутов Git для эффективного сравнения двоичных файлов. Вы делаете это, сообщая Git, как преобразовать ваши двоичные данные в текстовый формат, который можно сравнить с помощью обычного diff.

Во-первых, вы будете использовать эту технику для решения одной из самых досадных проблем, известных человечеству: контроль версий документов Microsoft Word. Все знают, что Word - самый ужасный редактор из всех, но, как ни странно, все еще используют его. Если вы хотите контролировать документы Word, вы можете поместить их в Git-репозиторий и делать коммиты время от времени; но что хорошего в этом? Если вы запускаете git diff нормально, вы видите только что-то вроде этого:

```console
$ git diff
diff --git a/chapter1.docx b/chapter1.docx
index 88839c4..4afcb7c 100644
Binary files a/chapter1.docx and b/chapter1.docx differ
```

Вы не можете напрямую сравнить две версии, если не проверите их и не отсканируете их вручную, верно? Оказывается, вы можете сделать это довольно хорошо, используя атрибуты Git. Поместите следующую строку в ваш файл `.gitattributes`:

```ini
*.docx diff=word
```

Это говорит Git, что любой файл, соответствующий этому шаблону (`.docx`), должен использовать фильтр «word», когда вы пытаетесь просмотреть diff, содержащий изменения. Что такое «word» фильтр? Вы должны настроить это. Здесь вы сконфигурируете Git для использования программы `docx2txt` для преобразования документов Word в читаемые текстовые файлы, которые затем будут правильно отображаться.

Во-первых, вам нужно установить `docx2txt`; Вы можете скачать его с [https://sourceforge.net/projects/docx2txt](https://sourceforge.net/projects/docx2txt). Следуйте инструкциям в файле `INSTALL`, чтобы поместить его туда, где его может найти ваша оболочка. Затем вы напишите скрипт-обертку для преобразования вывода в формат, ожидаемый Git. Создайте файл, который находится где-то на вашем пути, под названием `docx2txt`, и добавьте следующее содержимое:

```console
#!/bin/bash
docx2txt.pl "$1" -
```

Не забудьте `chmod a + x` этот файл. Наконец, вы можете настроить Git для использования этого скрипта:

```console
$ git config diff.word.textconv docx2txt
```

Теперь Git знает, что если он пытается выполнить различие между двумя снимками, и любой из файлов заканчивается на `.docx`, он должен запустить эти файлы через фильтр «word», который определен как программа« docx2txt». Это эффективно делает хорошие текстовые версии ваших файлов Word, прежде чем пытаться их различать.

Вот пример: Глава 1 этой книги была преобразована в формат Word и помещена в репозиторий Git. Затем был добавлен новый абзац. Вот что показывает `git diff`:

```console
$ git diff
diff --git a/chapter1.docx b/chapter1.docx
index 0b013ca..ba25db5 100644
--- a/chapter1.docx
+++ b/chapter1.docx
@@ -2,6 +2,7 @@
 This chapter will be about getting started with Git. We will begin at the beginning by explaining some background on version control tools, then move on to how to get Git running on your system and finally how to get it setup to start working with. At the end of this chapter you should understand why Git is around, why you should use it and you should be all setup to do so.
 1.1. About Version Control
 What is "version control", and why should you care? Version control is a system that records changes to a file or set of files over time so that you can recall specific versions later. For the examples in this book you will use software source code as the files being version controlled, though in reality you can do this with nearly any type of file on a computer.
+Testing: 1, 2, 3.
 If you are a graphic or web designer and want to keep every version of an image or layout (which you would most certainly want to), a Version Control System (VCS) is a very wise thing to use. It allows you to revert files back to a previous state, revert the entire project back to a previous state, compare changes over time, see who last modified something that might be causing a problem, who introduced an issue and when, and more. Using a VCS also generally means that if you screw things up or lose files, you can easily recover. In addition, you get all this for very little overhead.
 1.1.1. Local Version Control Systems
 Many people's version-control method of choice is to copy files into another directory (perhaps a time-stamped directory, if they're clever). This approach is very common because it is so simple, but it is also incredibly error prone. It is easy to forget which directory you're in and accidentally write to the wrong file or copy over files you don't mean to.
```

Git успешно и лаконично сообщает нам, что мы добавили строку «Testing: 1, 2, 3.», и это правильно. Он не идеален - изменения форматирования здесь не появятся - но это, безусловно, работает.

Еще одна интересная проблема, которую вы можете решить таким образом, заключается в разграничении файлов изображений. Один из способов сделать это - запустить файлы изображений через фильтр, который извлекает их EXIF-информацию - метаданные, записанные в большинстве форматов изображений. Если вы скачаете и установите программу `exiftool`, вы можете использовать ее для преобразования ваших изображений в текст о метаданных, поэтому, по крайней мере, diff покажет вам текстовое представление всех произошедших изменений. Поместите следующую строку в ваш файл `.gitattributes`:

```ini
*.png diff=exif
```

Настройте Git для использования этого инструмента:

```console
$ git config diff.exif.textconv exiftool
```

Если вы замените изображение в своем проекте и запустите `git diff`, вы увидите что-то вроде этого:

```diff
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
```

Вы можете легко увидеть, что размер файла и размеры изображения изменились.

### Расширение ключевого слова

Разработчики, использующие эти системы, часто запрашивают расширение ключевых слов в стиле SVN или CVS. Основная проблема с этим в Git состоит в том, что вы не можете изменить файл с информацией о коммите после того, как вы зафиксировали, потому что Git сначала проверяет суммы файла. Тем не менее, вы можете вставить текст в файл, когда он извлечен, и удалить его снова, прежде чем он будет добавлен в коммит. Атрибуты Git предлагают вам два способа сделать это.

Во-первых, вы можете автоматически ввести контрольную сумму SHA-1 большого двоичного объекта в поле `$Id$` в файле. Если вы установите этот атрибут для файла или набора файлов, то в следующий раз, когда вы извлечете эту ветку, Git заменит это поле на SHA-1 большого двоичного объекта. Важно отметить, что это не SHA-1 коммита, а сам блоб. Поместите следующую строку в ваш файл `.gitattributes`:

```ini
*.txt ident
```

Добавьте ссылку `$Id$` в тестовый файл:

```console
$ echo '$Id$' > test.txt
```

The next time you check out this file, Git injects the SHA-1 of the blob:

```console
$ rm test.txt
$ git checkout -- test.txt
$ cat test.txt
$Id: 42812b7653c7b88933f8a9d6cad0ca16714b9bb3 $
```

However, that result is of limited use. If you’ve used keyword substitution in CVS or Subversion, you can include a datestamp – the SHA-1 isn’t all that helpful, because it’s fairly random and you can’t tell if one SHA-1 is older or newer than another just by looking at them.

It turns out that you can write your own filters for doing substitutions in files on commit/checkout. These are called “clean” and “smudge” filters. In the`.gitattributes`file, you can set a filter for particular paths and then set up scripts that will process files just before they’re checked out (“smudge”, see[The “smudge” filter is run on checkout.](https://git-scm.com/book/en/v2/ch00/filters_a)) and just before they’re staged (“clean”, see[The “clean” filter is run when files are staged.](https://git-scm.com/book/en/v2/ch00/filters_b)). These filters can be set to do all sorts of fun things.

![](/images/8c34c1aeb3ddf12194cfa7ac92160491.png)

Figure 144. The “smudge” filter is run on checkout.

![The ``smudge'' filter is run on checkout.](/images/2b92285c3b797e606ca4792eb9ff5dde.png)Figure 144. The “smudge” filter is run on checkout.![The ``clean'' filter is run when files are staged.](/images/ac5b46b36c9200ccec5d88a7159350d9.png)

Figure 145. The “clean” filter is run when files are staged.

The original commit message for this feature gives a simple example of running all your C source code through the`indent`program before committing. You can set it up by setting the filter attribute in your`.gitattributes`file to filter`*.c`files with the “indent” filter:

```ini
*.c filter=indent
```

Then, tell Git what the “indent” filter does on smudge and clean:

```console
$ git config --global filter.indent.clean indent
$ git config --global filter.indent.smudge cat
```

In this case, when you commit files that match`*.c`, Git will run them through the indent program before it stages them and then run them through the`cat`program before it checks them back out onto disk. The`cat`program does essentially nothing: it spits out the same data that it comes in. This combination effectively filters all C source code files through`indent`before committing.

Another interesting example gets`$Date$`keyword expansion, RCS style. To do this properly, you need a small script that takes a filename, figures out the last commit date for this project, and inserts the date into the file. Here is a small Ruby script that does that:

```ruby
#! /usr/bin/env ruby
data = STDIN.read
last_date = `git log --pretty=format:"%ad" -1`
puts data.gsub('$Date$', '$Date: ' + last_date.to_s + '$')
```

All the script does is get the latest commit date from the`git log`command, stick that into any`$Date$`strings it sees in stdin, and print the results – it should be simple to do in whatever language you’re most comfortable in. You can name this file`expand_date`and put it in your path. Now, you need to set up a filter in Git (call it`dater`) and tell it to use your`expand_date`filter to smudge the files on checkout. You’ll use a Perl expression to clean that up on commit:

```console
$ git config filter.dater.smudge expand_date
$ git config filter.dater.clean 'perl -pe "s/\\\$Date[^\\\$]*\\\$/\\\$Date\\\$/"'
```

This Perl snippet strips out anything it sees in a`$Date$`string, to get back to where you started. Now that your filter is ready, you can test it by setting up a Git attribute for that file that engages the new filter and creating a file with your`$Date$`keyword:

```ini
date*.txt filter=dater
```

```console
$ echo '# $Date$' > date_test.txt
```

If you commit those changes and check out the file again, you see the keyword properly substituted:

```console
$ git add date_test.txt .gitattributes
$ git commit -m "Testing date expansion in Git"
$ rm date_test.txt
$ git checkout date_test.txt
$ cat date_test.txt
# $Date: Tue Apr 21 07:26:52 2009 -0700$
```

You can see how powerful this technique can be for customized applications. You have to be careful, though, because the`.gitattributes`file is committed and passed around with the project, but the driver (in this case,`dater`) isn’t, so it won’t work everywhere. When you design these filters, they should be able to fail gracefully and have the project still work properly.

### Exporting Your Repository

Git attribute data also allows you to do some interesting things when exporting an archive of your project.

#### `export-ignore`

You can tell Git not to export certain files or directories when generating an archive. If there is a subdirectory or file that you don’t want to include in your archive file but that you do want checked into your project, you can determine those files via the`export-ignore`attribute.

For example, say you have some test files in a`test/`subdirectory, and it doesn’t make sense to include them in the tarball export of your project. You can add the following line to your Git attributes file:

```ini
test/ export-ignore
```

Now, when you run`git archive`to create a tarball of your project, that directory won’t be included in the archive.

#### `export-subst`

When exporting files for deployment you can apply`git log`'s formatting and keyword-expansion processing to selected portions of files marked with the`export-subst`attribute.

For instance, if you want to include a file named`LAST_COMMIT`in your project, and have metadata about the last commit automatically injected into it when`git archive`runs, you can for example set up your`.gitattributes`and`LAST_COMMIT`files like this:

```ini
LAST_COMMIT export-subst
```

```console
$ echo 'Last commit date: $Format:%cd by %aN$' > LAST_COMMIT
$ git add LAST_COMMIT .gitattributes
$ git commit -am 'adding LAST_COMMIT file for archives'
```

When you run`git archive`, the contents of the archived file will look like this:

```console
$ git archive HEAD | tar xCf ../deployment-testing -
$ cat ../deployment-testing/LAST_COMMIT
Last commit date: Tue Apr 21 08:38:48 2009 -0700 by Scott Chacon
```

The substitutions can include for example the commit message and any`git notes`, and`git log`can do simple word wrapping:

```console
$ echo '$Format:Last commit: %h by %aN at %cd%n%+w(76,6,9)%B$' > LAST_COMMIT
$ git commit -am 'export-subst uses git log'\''s custom formatter

git archive uses git log'\''s `pretty=format:` processor
directly, and strips the surrounding `$Format:` and `$`
markup from the output.
'
$ git archive @ | tar xfO - LAST_COMMIT
Last commit: 312ccc8 by Jim Hill at Fri May 8 09:14:04 2015 -0700
       export-subst uses git log's custom formatter

         git archive uses git log's `pretty=format:` processor directly, and
         strips the surrounding `$Format:` and `$` markup from the output.
```

The resulting archive is suitable for deployment work, but like any exported archive it isn’t suitable for further development work.

### Merge Strategies

You can also use Git attributes to tell Git to use different merge strategies for specific files in your project. One very useful option is to tell Git to not try to merge specific files when they have conflicts, but rather to use your side of the merge over someone else’s.

This is helpful if a branch in your project has diverged or is specialized, but you want to be able to merge changes back in from it, and you want to ignore certain files. Say you have a database settings file called`database.xml`that is different in two branches, and you want to merge in your other branch without messing up the database file. You can set up an attribute like this:

```ini
database.xml merge=ours
```

And then define a dummy`ours`merge strategy with:

```console
$ git config --global merge.ours.driver true
```

If you merge in the other branch, instead of having merge conflicts with the`database.xml`file, you see something like this:

```console
$ git merge topic
Auto-merging database.xml
Merge made by recursive.
```

In this case,`database.xml`stays at whatever version you originally had.

**********
[gitattributes](/tags/gitattributes.md)
[атрибуты](/tags/%D0%B0%D1%82%D1%80%D0%B8%D0%B1%D1%83%D1%82%D1%8B.md)
