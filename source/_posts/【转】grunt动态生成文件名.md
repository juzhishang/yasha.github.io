title: 【转】grunt动态生成文件名
date: 2015-12-09 16:44:13
tags:
 - grunt
---
动态生成文件名

expand 设置为true打开以下选项

cwd 所有src指定的文件相对于这个属性指定的路径

src 要匹配的路径，相对与cwd

dest 生成的目标路径前缀

ext 替换所有生成的目标文件后缀为这个属性

flatten 删除所有生成的dest的路径部分

rename 一个函数，接受匹配到的文件名，和匹配的目标位置，返回一个新的目标路径
```javascript
grunt.initConfig({
    minify: {
        dynamic_mappings: {
            // Grunt will search for "**/?.js" under "lib/" when the "minify" task runs 
            files: [
                {
                    expand: true,     // Enable dynamic expansion.
                    cwd: 'lib/'       // Src matches are relative to this path.
                    src: ['**/?.js'], // Actual pattern(s) to match.
                    dest: 'build/',   // Destination path prefix.
                    ext: '.min.js',   // Dest filepaths will have this extension.
                }
            ]
        }
    }
});
```
原文：http://www.hulufei.com/post/grunt-introduction