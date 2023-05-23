## Git

### 邮箱使用错误修改

修改WRONG_EMAIL为旧邮箱,修改NEW_EMAIL为新邮箱，NEW_NAME为新用户名

```
git filter-branch --env-filter '
WRONG_EMAIL="old@gmail.com"
NEW_NAME="new_name"
NEW_EMAIL="new@gmail.com"

if [ "$GIT_COMMITTER_EMAIL" = "$WRONG_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$NEW_NAME"
    export GIT_COMMITTER_EMAIL="$NEW_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$WRONG_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$NEW_NAME"
    export GIT_AUTHOR_EMAIL="$NEW_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

