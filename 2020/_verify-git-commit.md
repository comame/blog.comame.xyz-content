ただの備忘録

```
$ gpg --full-generate-key
$ gpg --armor --export <hash>

$ git config --global gpg.program gpg
$ git config --global user.signingkey <hash>
$ git config --global commit.gpgsign true
```

秘密鍵にパスフレーズを設定するとコミットができないのはなぜ...?
