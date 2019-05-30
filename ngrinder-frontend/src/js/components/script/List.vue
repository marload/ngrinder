<template>
    <div class="container">
        <vue-headful title="Script"></vue-headful>
        <div class="script-img-unit"></div>

        <search-bar :scripts="scripts" :currentPath="currentPath"></search-bar>

        <table class="table table-striped table-bordered ellipsis dataTable">
            <colgroup>
                <col width="30">
                <col width="32">
                <col width="230">
                <col>
                <col width="180">
                <col width="80">
                <col width="80">
                <col width="80">
            </colgroup>
            <thead>
            <tr>
                <th><input type="checkbox" class="checkbox" value=""></th>
                <th class="no-click">
                    <router-link :to="baseDirectory" target="_self">
                        <img src="/img/up_folder.png"/>
                    </router-link>
                </th>
                <th v-text="i18n('script.list.name')"></th>
                <th class="no-click" v-text="i18n('script.list.commit')"></th>
                <th v-text="i18n('script.list.lastDate')"></th>
                <th v-text="i18n('script.list.revision')"></th>
                <th v-text="i18n('script.list.size')"></th>
                <th class="no-click" v-text="i18n('script.list.download')"></th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="script in scripts">
                <td>
                    <input v-if="script.fileName !== '..'" type="checkbox" class="checkbox" v-model="script.checked">
                </td>
                <td>
                    <i v-if="isEditable(script.fileType , script.path)" class="icon-file"></i>
                    <i v-else-if="script.fileType === 'DIR'" class="icon-folder-open"></i>
                    <i v-else class="icon-briefcase"></i>
                </td>
                <td class="ellipsis">
                    <a v-if="isEditable(script.fileType , script.path)" :href="`/script/detail/${script.path}`"
                       target="_self" :title="script.path" v-text="script.fileName"></a>
                    <router-link v-else-if="script.fileType === 'DIR'" :to="`/script/list/${script.path}`"
                                 target="_self" :title="script.path" v-text="script.fileName"></router-link>
                    <a v-else :href="`/script/download/${script.path}`" target="_blank" :title="script.path"
                       v-text="script.fileName"></a>
                </td>
                <td class="ellipsis" :title="script.description" v-text="script.description"></td>
                <td><span v-if="!!script.lastModifiedDate">{{ script.lastModifiedDate | dateFormat('YYYY-MM-DD HH:mm') }}</span></td>
                <td v-text="script.revision"></td>
                <td><span v-text="script.fileType !== 'DIR' ? getFileSize(script.fileSize) : ''"></span></td>
                <td class="center">
                    <i v-if="script.fileType !== 'DIR'" class="pointer-cursor icon-download-alt script-download" :spath="script.path" v-on:click="downloadScript(script.path)"></i>
                </td>
            </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
    import { Component, Watch } from 'vue-property-decorator';
    import VueHeadful from 'vue-headful';
    import FileType from '../../common/file-type';

    import Base from '../Base.vue';
    import SearchBar from './SearchBar.vue';

    @Component({
        name: 'scriptList',
        components: {VueHeadful, SearchBar},
    })
    export default class ScriptList extends Base {
        scripts = [];

        mounted() {
            this.refreshScriptList();
            this.$EventBus.$on(this.$Event.REFRESH_SCRIPT_LIST, this.refreshScriptList);
        }

        refreshScriptList() {
            this.$http.get(`/script/api/list/${this.currentPath}`, {
                params: {
                    user: this.currentUser
                }
            }).then(res => {
                this.scripts.splice(0);
                this.scripts.push(
                    ...res.data.map(script => {
                        script.checked = false;
                        return script
                    })
                    .map(script => {
                        script.fileName = this.getFileName(script.path);
                        return script
                    }));
            });
        }

        get currentPath() {
            return removePrependedSlash(this.$route.path).replace('/script', '').replace('/list', '').replace('/', '');
        }

        get baseDirectory() {
            if (this.currentPath === '' || this.currentPath.lastIndexOf('/') < 0) {
                return '/script/list'
            }
            return '/script/list/' + this.currentPath.slice(0, this.currentPath.lastIndexOf('/'));
        }

        @Watch('$route')
        watchRoute() {
            this.refreshScriptList();
        }

        downloadScript(path) {
            window.location.href = `/script/download/${path}`;
        }

        getFileSize(size) {
            return this.formatNumber(( size / 1024 ), 2);
        }

        isEditable(type, path) {
            if (type) {
                return FileType.FileTypeEnum[type].isEditable();
            } else {
                const ext = this.getFileExtension(path);
                return FileType.getFileTypeByExtension(ext).isEditable();
            }
        }

        getFileName(filepath) {
            if (filepath) {
                const lastIndex = this.indexOfLastSeparator(filepath);
                return filepath.substring(lastIndex + 1);
            } else {
                return '';
            }
        }

        indexOfLastSeparator(filepath) {
            const lastUnixPos = filepath.lastIndexOf('/');
            const lastWindowsPos = filepath.lastIndexOf('\\');
            return Math.max(lastUnixPos, lastWindowsPos);
        }

        getFileExtension(filepath) {
            if (!filepath) {
                return '';
            } else {
                let extensionPos = filepath.lastIndexOf('.');
                const lastSeparator = this.indexOfLastSeparator(filepath);
                extensionPos = lastSeparator > extensionPos ? -1 : extensionPos;
                return extensionPos === -1 ? "" : filepath.substring(extensionPos + 1);
            }
        }

        formatNumber(number, decimal) {
            if (number === 0) return 0;
            const n = (number + '');
            const parts = n.toString().split(".");
            return parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ",") + (parts[1] ? "." + ( parts[1].length > decimal ? parts[1].substring(0, decimal) : parts[1] ) : "");
        }
    }

    const removePrependedSlash = path => path.endsWith('/') ? path.slice(0, path.length - 1) : path;
</script>

<style scoped>
    .script-img-unit {
        background-image: url('/img/bg_script_banner_en.png');
        height: 110px;
        padding: 0;
        margin-top: 0;
    }

    .table {
        font-size: 12px;
    }
</style>