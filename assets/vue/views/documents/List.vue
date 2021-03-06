<template>
  <span class="documents-list">
    <Toolbar
      :handle-add="addHandler"
      :handle-add-document="addDocumentHandler"
      :handle-upload-document="uploadDocumentHandler"

      :filters="filters"
      :on-send-filter="onSendFilter"
      :reset-filter="resetFilter"
    />

    <br>
    <b-row class="text-center">
      <b-col>
        <form class="form-inline">
          <div class="form-group mb-2">
            <b-form-select
              id="perPageSelect"
              v-model="options.itemsPerPage"
              size="sm"
              :options="pageOptions"
              @input="onUpdateOptions(options)"
            />
          </div>
        </form>
      </b-col>
      <b-col />
      <b-col>
        <b-pagination
          v-model="options.page"
          :disabled.sync="isLoading"
          :total-rows="totalItems"
          :per-page="options.itemsPerPage"
          aria-controls="documents"
          align="right"
          size="sm"
          @input="onUpdateOptions(options)"
        />
      </b-col>
    </b-row>
    <b-row>
      <b-col>
        <b-table
          id="documents"
          striped
          hover
          no-local-sorting
          responsive="sm"

          :per-page="0"
          :fields="fields"
          :items="items"
          :current-page.sync="options.page"

          :sort-by.sync="options.sortBy"
          :sort-desc.sync="options.sortDesc"

          :busy.sync="isLoading"
          :filters="filters"
          primary-key="iid"
          @sort-changed="sortingChanged"
        >
          <template v-slot:table-busy>
            <div class="text-center my-2">
              <b-spinner class="align-middle" />
              <strong>{{ $t('Loading ...') }}</strong>
            </div>
          </template>

          <template
            v-slot:cell(resourceNode.title)="row"
          >
            <div v-if="row.item['resourceNode']['resourceFile']">
              <a
                data-fancybox="gallery"
                :href="row.item['contentUrl'] "
              >
                <ResourceFileIcon :file="row.item['resourceNode']['resourceFile']" />

                {{ row.item['resourceNode']['title'] }}
              </a>
            </div>
            <div v-else>
              <a @click="handleClick(row.item)">
                <font-awesome-icon icon="folder" />
                {{ row.item['resourceNode']['title'] }}
              </a>
            </div>
          </template>

          <template
            v-slot:cell(resourceNode.updatedAt)="row"
          >
            {{ row.item.resourceNode.updatedAt | moment("from", "now") }}
          </template>

          <template
            v-slot:cell(resourceNode.resourceFile.size)="row"
          >
            <span
              v-if="row.item['resourceNode']['resourceFile']"
            >
              {{ row.item.resourceNode.resourceFile.size | prettyBytes }}
            </span>

          </template>

          <template
            v-slot:cell(action)="row"
          >
            <ActionCell
              slot="action"
              :row="row"
              :handle-show="() => showHandler(row.item)"
              :handle-edit="() => editHandler(row.item)"
              :handle-delete="() => deleteHandler(row.item)"
            />
          </template>
        </b-table>
      </b-col>
    </b-row>
  </span>
</template>

<script>
import { mapActions, mapGetters } from 'vuex';
import { mapFields } from 'vuex-map-fields';
import ListMixin from '../../mixins/ListMixin';
import ActionCell from '../../components/ActionCell';
import Toolbar from '../../components/Toolbar';
import ResourceFileIcon from './ResourceFileIcon';


export default {
    name: 'DocumentsList',
    servicePrefix: 'Documents',
    components: {
      Toolbar,
      ActionCell,
      ResourceFileIcon
    },
    mixins: [ListMixin],
    data() {
        return {
          sortBy: 'title',
          sortDesc: false,
          fields: [
                {label: 'Title', key: 'resourceNode.title', sortable: true},
                {label: 'Modified', key: 'resourceNode.updatedAt', sortable: true},
                {label: 'Size', key: 'resourceNode.resourceFile.size', sortable: true},
                {label: 'Actions', key: 'action', sortable: false}
            ],
            selected: [],
            pageOptions: [5, 10, 15, 20],
        };
    },
    created() {
        let nodeId = this.$route.params['node'];
        this.findResourceNode('/api/resource_nodes/'+ nodeId);
        this.onUpdateOptions(this.options);
    },
    computed: {
        // From crud.js list function
        ...mapGetters('documents', {
            items: 'list'
        }),
        ...mapGetters('resourcenode', {
            resourceNode: 'getResourceNode'
        }),
        // From ListMixin
        ...mapFields('documents', {
            deletedItem: 'deleted',
            error: 'error',
            isLoading: 'isLoading',
            resetList: 'resetList',
            totalItems: 'totalItems',
            view: 'view'
        }),
    },
    methods: {
        sortingChanged(ctx) {
          this.options.sortDesc = ctx.sortDesc;
          this.options.sortBy = ctx.sortBy;
          this.onUpdateOptions(this.options);
          // ctx.sortBy   ==> Field key for sorting by (or null for no sorting)
          // ctx.sortDesc ==> true if sorting descending, false otherwise
        },
        // From ListMixin
        ...mapActions('documents', {
            getPage: 'fetchAll',
            deleteItem: 'del'
        }),
        ...mapActions('resourcenode', {
            findResourceNode: 'findResourceNode',
        }),
    }
};
</script>
