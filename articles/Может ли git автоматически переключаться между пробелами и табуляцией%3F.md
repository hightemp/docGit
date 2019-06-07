# Может ли git автоматически переключаться между пробелами и табуляцией?

## Очень полезная информация для всех, кто использует GitHub (или другой подобный сервис)

`~/.gitconfig`

```
[filter "tabspace"]
    smudge = unexpand --tabs=4 --first-only
    clean = expand --tabs=4 --initial
[filter "tabspace2"]
    smudge = unexpand --tabs=2 --first-only
    clean = expand --tabs=2 --initial

```

Тогда у меня есть два файла: `attributes`

```
*.js  filter=tabspace
*.html  filter=tabspace
*.css  filter=tabspace
*.json  filter=tabspace

```

а также `attributes2`

```
*.js  filter=tabspace2
*.html  filter=tabspace2
*.css  filter=tabspace2
*.json  filter=tabspace2

```

## Работа над личными проектами

```
mkdir project
cd project
git init
cp ~/path/to/attributes .git/info/

```

Таким образом, когда вы, наконец, поместите свою работу на github, она не будет выглядеть глупо в представлении кода с `8 пробелами`, что является поведением по умолчанию во всех браузерах.

## Вклад в другие проекты

```
mkdir project
cd project
git init
cp ~/path/to/attributes2 .git/info/attributes
git remote add origin git@github.com:some/repo.git
git pull origin branch

```

Таким образом, вы можете работать с обычными вкладками в проектах с отступом 2.

Конечно, вы можете написать аналогичное решение для преобразования из «4 пробелов в 2 пробелов», что имеет место, если вы хотите участвовать в проектах, опубликованных мной, и вы склонны использовать 2 пробела при разработке.