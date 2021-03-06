<template>

    <div>
        <div class="box"
            v-if="!summary">
            <div class="columns">
                <div class="column is-one-third">
                    <vue-select :options="importTypes"
                        v-model="importType"
                        @input="getTemplate"
                        key-map="string"
                        ref="importTypeSelect">
                    </vue-select>
                </div>
                <div class="column has-text-centered has-padding-medium"
                    v-if="importType">
                    <file-uploader class="animated fadeIn"
                        v-if="!template"
                        :url="templateLink"
                        @upload-start="loadingTemplate=true"
                        @upload-successful="template = $event;loadingTemplate = false"
                        @upload-error="loadingTemplate=false">
                        <a slot="upload-button"
                            class="button is-info">
                            <span>{{ __('Upload Template') }}</span>
                            <span class="icon is-small">
                                <fa icon="upload"></fa>
                            </span>
                        </a>
                    </file-uploader>
                    <a class="button is-info animated fadeIn has-margin-right-small"
                        v-if="template"
                        v-tooltip="template.original_name"
                        :href="downloadLink">
                        <span>{{ __('Download Template') }}</span>
                        <span class="icon is-small">
                            <fa icon="download"></fa>
                        </span>
                    </a>
                    <a class="button is-danger animated fadeIn"
                        v-if="template"
                        @click="showModal = true">
                        <span>{{ __('Delete Template') }}</span>
                        <span class="icon is-small">
                            <fa icon="trash-alt"></fa>
                        </span>
                    </a>
                </div>
                <div class="column animated fadeIn"
                    v-if="importType">
                    <file-uploader class="is-pulled-right"
                        @upload-start=" importing = true"
                        @upload-successful="summary = $event;importing = false"
                        @upload-error="importing = false;importType = null"
                        :url="importLink">
                        <a slot="upload-button"
                            :class="['button is-success', { 'is-loading': importing }]">
                            <span>{{ __('Start Import') }}</span>
                            <span class="icon is-small">
                                <fa icon="upload"></fa>
                            </span>
                        </a>
                    </file-uploader>
                    <div class="is-clearfix"></div>
                </div>
            </div>
        </div>
        <vue-table class="animated fadeIn"
            :path="path"
            :i18n="__"
            id="imports-table"
            :custom-render="customRender"
            @get-summary="getSummary"
            v-if="!summary">
        </vue-table>
        <div class="columns"
            v-if="summary">
            <div class="column is-half-tablet is-one-third-widescreen is-one-quarter-fullhd animated bounceInLeft">
                <nav class="box panel is-paddingless">
                    <p class="panel-heading">
                        {{ __('Import Summary') }}
                    </p>
                    <a class="panel-block">
                        <span class="panel-icon has-text-info">
                            <fa icon="file-excel"></fa>
                        </span>
                        {{ __('File') }}:&emsp;<span class="has-text-info">{{ summary.filename }}</span>
                    </a>
                    <a class="panel-block">
                        <span class="panel-icon has-text-info">
                            <fa icon="calendar-alt"></fa>
                        </span>
                        {{ __('Date') }}:&emsp;<span class="has-text-info">{{ summary.date }}</span>
                    </a>
                    <a class="panel-block">
                        <span class="panel-icon has-text-info">
                            <fa icon="clock"></fa>
                        </span>
                        {{ __('Time') }}:&emsp;<span class="has-text-info">{{ summary.time }}</span>
                    </a>
                    <a class="panel-block">
                        <span class="panel-icon has-text-success">
                            <fa icon="check"></fa>
                        </span>
                        {{ __('Imported Entries') }}:&emsp;<span class="has-text-success">{{ summary.successful }}</span>
                    </a>
                    <a class="panel-block">
                        <span class="panel-icon has-text-danger">
                            <fa icon="times"></fa>
                        </span>
                        {{ __('Issues') }}:&emsp;<span class="has-text-danger">{{ summary.issues }}</span>
                    </a>
                    <div class="panel-block">
                        <button class="button is-info is-outlined is-fullwidth"
                            @click="summary=null">
                            {{ __("Back") }}
                        </button>
                    </div>
                </nav>
            </div>
            <div class="column is-half-tablet is-two-thirds-widescreen is-three-quarters-fullhd animated bounceInRight">
                <card :icon="icon"
                    :title="__('Issues')"
                    removable
                    @remove="summary = null"
                    v-if="summary.issues">
                    <tabs class="has-padding-medium"
                        alignment="right">
                        <tab v-for="(issues, category) in summary.structureIssues"
                            :key="category"
                            :id="category">
                            <ul class="issues has-margin-left-large">
                                <li v-for="(issue, index) in issues"
                                    :key="index">
                                    <span>
                                        <b class="has-text-danger">{{ issue }}</b>
                                    </span>
                                </li>
                            </ul>
                        </tab>
                        <tab v-for="(sheetIssues, sheet) in summary.contentIssues"
                            :key="sheet"
                            :id="sheet">
                            <tabs>
                                <tab v-for="(issues, category) in sheetIssues"
                                    :key="category"
                                    :id="category">
                                    <paginate :list="issues">
                                        <template slot-scope="props">
                                            <h5 class="title is-5 has-text-centered">{{ __('Issues') }}</h5>
                                            <ul class="issues has-margin-left-large">
                                                <li v-for="(issue, index) in props.list"
                                                    :key="index">
                                                    <span v-if="issue.column">
                                                        {{ __("Column") }}: <b class="has-text-warning">{{ issue.column }}</b>
                                                    </span>
                                                    <span v-if="issue.rowNumber">
                                                        {{ __("Line") }}: <b class="has-text-warning">{{ issue.rowNumber }}</b>
                                                    </span>
                                                    <span v-if="issue.value">
                                                        {{ __("Value") }}: <b class="has-text-danger">{{ issue.value }}</b>
                                                    </span>
                                                </li>
                                            </ul>
                                        </template>
                                    </paginate>
                                </tab>
                            </tabs>
                        </tab>
                    </tabs>
                </card>
            </div>
        </div>
        <modal :show="showModal"
            @cancel="showModal = false"
            @commit="deleteTemplate(template.id)">
        </modal>
    </div>

</template>

<script>

import { mapGetters } from 'vuex';
import { VTooltip } from 'v-tooltip';
import fontawesome from '@fortawesome/fontawesome';
import {
    faUpload, faDownload, faTrashAlt, faFileExcel,
    faCalendarAlt, faClock, faCheck, faTimes, faBook,
} from '@fortawesome/fontawesome-free-solid/shakable.es';
import VueSelect from '../../components/enso/select/VueSelect.vue';
import VueTable from '../../components/enso/vuedatatable/VueTable.vue';
import FileUploader from '../../components/enso/fileuploader/FileUploader.vue';
import Modal from '../../components/enso/bulma/Modal.vue';
import Card from '../../components/enso/bulma/Card.vue';
import Overlay from '../../components/enso/bulma/Overlay.vue';
import Paginate from '../../components/enso/bulma/Paginate.vue';
import Tabs from '../../components/enso/bulma/Tabs.vue';
import Tab from '../../components/enso/bulma/Tab.vue';

fontawesome.library.add([
    faUpload, faDownload, faTrashAlt, faFileExcel,
    faCalendarAlt, faClock, faCheck, faTimes, faBook,
]);

export default {
    components: {
        VueSelect, VueTable, FileUploader, Card, Modal, Overlay, Tabs, Tab, Paginate,
    },

    directives: { tooltip: VTooltip },

    computed: {
        ...mapGetters('locale', ['__']),
        templateLink() {
            return route('import.uploadTemplate', this.importType, false);
        },
        downloadLink() {
            return route('import.downloadTemplate', this.template.id, false);
        },
        importLink() {
            return route('import.run', this.importType, false);
        },
        icon() {
            return faBook;
        },
    },

    data() {
        return {
            path: route('import.initTable', [], false),
            importType: null,
            summary: null,
            template: null,
            showModal: false,
            loadingTemplate: false,
            importing: false,
            importTypes: {},
        };
    },

    created() {
        axios.get(route('import.index', [], false)).then(({ data }) => {
            this.importTypes = data.importTypes;
        }).catch(error => this.handleError(error));
    },

    methods: {
        getTemplate() {
            if (!this.importType) {
                return;
            }

            this.loadingTemplate = true;

            axios.get(route('import.getTemplate', this.importType, false)).then(({ data }) => {
                this.template = data;
                this.loadingTemplate = false;
            }).catch((error) => {
                this.loadingTemplate = false;
                this.handleError(error);
            });
        },
        deleteTemplate(id) {
            this.loadingTemplate = true;
            axios.delete(route('import.deleteTemplate', id, false)).then(({ data }) => {
                this.template = null;
                this.showModal = false;
                this.$toastr.success(data.message);
                this.loadingTemplate = false;
            }).catch((error) => {
                this.showModal = false;
                this.loadingTemplate = false;
                this.handleError(error);
            });
        },
        getSummary(row) {
            this.loading = true;

            axios.get(route('import.getSummary', row.dtRowId, false)).then(({ data }) => {
                this.loading = false;

                if (data.issues === 0) {
                    this.$toastr.info('The import has no issues');
                    return;
                }

                this.summary = data;
            }).catch((error) => {
                this.loading = false;
                this.handleError(error);
            });
        },
        customRender(row, column) {
            switch (column.name) {
            case 'successful':
                return `<b class="has-text-success">${row[column.name]}</b>`;
            case 'issues':
                return `<b class="has-text-danger">${row[column.name]}</b>`;
            default:
                this.$toastr.warning(`render for column ${column.name} is not defined.`);
                return row[column.name];
            }
        },
    },
};

</script>

<style lang="scss" scoped>

    ul.issues {
        list-style-type: square;

        li {
            cursor: pointer;
            width: fit-content;
            padding: 0.2em;

            &:hover {
                background: lightgray;
            }
        }
    }

</style>
