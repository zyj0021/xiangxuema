<template>
    <div v-show="$root.aIndex>=0" id="editor">
        <div id="editorContainer"></div>
    </div>
</template>
<script>
    //todo:图片放大缩小需要按比例
    //todo:公式插件有问题
    const fs = nw.require('fs');
    const path = nw.require('path');
    export default {
        data() {
            return {

            }
        },
        computed: {
            article() {
                return this.$root.a[this.$root.aIndex];
            }
        },
        methods: {
            initEditor() {
                var editor = window.UE.getEditor('editorContainer');
                var self = this;
                editor.addListener("ready", () => {
                    self.hookImgInsert();
                    self.hookImgRemove();
                    self.hookSaveKeyEvent();
                    self.hookContentChange();
                    if (self.article) {
                        window.editorContentReady(self.article.id);
                    }
                });
            },
            hookContentChange() {
                var self = this;
                var subContent = document.getElementById("ueditor_0").contentWindow.document;
                subContent.oninput = function (e) {
                    self.$root.needSave.c = true;
                }
            },
            hookSaveKeyEvent() {
                var self = this;
                window.saveArticleKeyEvent = function () {
                    if (!self.$root.needSave.c) {
                        return;
                    }
                    self.$root.saveContent();
                };
            },
            hookContentReady() {
                window.editorContentReady = function (articleId) {
                    var aPath = path.join(window.nw.App.dataPath, "/xxm/" + articleId + "/a.data");
                    var content = fs.readFileSync(aPath, {
                        encoding: 'utf8'
                    });
                    if (UE.instants.ueditorInstant0 && UE.instants.ueditorInstant0.isReady) {
                        window.UE.instants.ueditorInstant0.setContent(content);
                    }
                }
            },
            hookImgRemove() {
                var self = this;
                var editorDocument = document.getElementById("ueditor_0").contentWindow.document;
                var observer = new MutationObserver(records => {
                    self.$root.needSave.c = true;
                    records.forEach((item, index) => {
                        if (item.removedNodes.length > 0 && item.removedNodes[0].tagName ==
                            "IMG") {
                            var path = decodeURI(item.removedNodes[0].src.substr(7));
                            fs.unlink(path, err => {
                                if (err) console.log(err);
                            });
                        }
                    });
                });
                observer.observe(editorDocument, {
                    childList: true,
                    subtree: true
                });
            },
            hookImgInsert() {
                var self = this;
                window.editorImgInsert = function (file) {
                    var basePath = path.join(window.nw.App.dataPath, "/xxm/" + self.article.id);
                    var id = "img" + new Date().getTime();
                    var name = path.join(basePath, id + '.' + file.name.split('.').last());
                    var fr = new FileReader();
                    fr.onload = () => {
                        if (fr.readyState == 2) {
                            var buffer = new Buffer(fr.result);
                            fs.writeFileSync(name, buffer);
                            var imgDom = '<img id="' + id + '" src="file://' + name + '" />';
                            window.UE.instants.ueditorInstant0.execCommand("inserthtml", imgDom);
                        }
                    };
                    fr.readAsArrayBuffer(file);
                }
            }
        },
        created() {
            this.hookContentReady();
        },
        mounted() {
            this.initEditor();
        }
    }
</script>
<style scoped lang="scss">

</style>