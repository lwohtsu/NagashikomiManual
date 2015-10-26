以下は通常の表のサンプルです。

|ファイル・フォルダ名|説明
|--|--
|cssjs| プレビュー用のCSSとJavaScriptファイルを入れておくフォルダ（P.■■参照）。
|doc| Markdown原稿を入れるフォルダだが、リネームしてもいいし、このフォルダ外に原稿を置いても問題ない。
|Gruntfile.js| Gruntの設定ファイル。
|indesign-plugins | InDesign用のプラグイン（P.■■参照）。インストール後は削除しても問題ない。
|kousei-sjis | Just-Rightという校正ツールを使用するために、Shift JISに変換したHTMLファイルを保存しておくフォルダ。削除しても問題ない。
|mytemplate.html | プレビュー用HTMLの元になるテンプレートファイル。CSSへのリンクなどを記述しておく（P.■■参照）。
|node_modules | Gruntやプラグインがインストールされているフォルダ。
|README.html〜xml | 使用説明ファイル。削除しても問題ない。
|sample | テスト用のファイル。削除しても問題ない。


以下はソースコードのサンプルです。

```javascript
module.exports = function(grunt) {
    grunt.initConfig({
        markdown: {
            all: {
                files: [
                    {
                        expand: true,
                        src: ['**/*.md', '!node_modules/**/*.md', '!**/ignore/**/*.md'],
                        ext: '.html'
                    }
                ],
                options: {
                    template: 'mytemplate.html',
                    postCompile: function(src, context) {
                        return src + "<script src='http://localhost:35732/livereload.js'></script>\n";
                    },
                    markdownOptions: {
                        gfm: true,
                        highlight: 'manual'
                    }
                }
            }
        },
        watch:{
            md: {
                files: ['**/*.md', '!**/ignore/**/*.*'],
                tasks: ['markdown'],
            },
            html: {
                files: ['**/*.html', '!kousei-sjis/**/*.html', '!**/ignore/**/*.html'],
                tasks: [ ],
                options: {
                    livereload: 35732
                }
            },
        },
        replace:{
            example: {
                expand: true,
                src: ['**/*.html', '!node_modules/**/*.html', '!kousei-sjis/**/*.html', '!**/ignore/**/*.html', '!**/mytemplate.html'],
                dest: 'kousei-sjis/',
                replacements: [{
                    from:  '<meta charset="utf-8">',
                    to: '<meta charset="sjis">'
                },
                {
                    from: /<script.*<\/script>/g, 
                    to:'<!---->'
                },
                {
                    from: /<link.*>/g, 
                    to:'<!---->'
                }
                ]
            }
        },
        utf8tosjis:{
            dist:{
                expand: true,
                src: ['kousei-sjis/**/*.html', '!**/mytemplate.html', '!**/ignore/**/*.html'],
                overwrite: true,
            }
        },
        html2indtag: {
            dist: {
                expand: true,
                src: ['**/*.html', '!node_modules/**/*.html', '!mytemplate.html', '!kousei-sjis/**/*.html', '!**/ignore/**/*.html'],
                ext: '.xml'
            },
            options: {
                //見出しが目立ちにくい場合は、trueにすると記号が入る
                midashialert: true
            }
        }
    });
 
    grunt.loadNpmTasks('grunt-utf8tosjis');
    grunt.loadNpmTasks('grunt-markdown');
    grunt.loadNpmTasks('grunt-contrib-watch');
    grunt.loadNpmTasks('grunt-text-replace');
    grunt.loadNpmTasks('grunt-html2indtag');
 
    grunt.registerTask("default", ["markdown", "html2indtag", "watch"]);
    grunt.registerTask("kousei", ["markdown", "replace", "utf8tosjis"/*, "watch"*/]);
    grunt.registerTask("xml", ["markdown", "html2indtag"]);
};
```