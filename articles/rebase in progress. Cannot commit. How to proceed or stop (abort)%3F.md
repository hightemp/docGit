# rebase in progress. Cannot commit. How to proceed or stop (abort)?

Rebase не происходит в фоновом режиме. «rebase in progress» означает, что вы начали перебазирование, и перебазировка была прервана из-за конфликта. Вы должны возобновить rebase (`git rebase --continue`) или прервать его (` git rebase --abort`).

Как показывает сообщение об ошибке из `git rebase --continue`, вы попросили git применить патч, который приводит к пустому патчу. Скорее всего, это означает, что патч уже был применен, и вы хотите удалить его с помощью `git rebase --skip`.

**********
[rebase](/tags/rebase.md)
