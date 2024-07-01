# pink-plus README

## Updating tagged colors

I've tagged colors that should be the same for different components using comments. If you want to replace them all at once, use the VSCode search function with the following search regexp and replace:

```regexp
("#)([0-9a-fA-F]{3}|[0-9a-fA-F]{6})(",? // #tag-name$)
```

```regexp
$1new-color$3
```
