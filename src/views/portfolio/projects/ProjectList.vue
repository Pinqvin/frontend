<template>
  <div class="animated fadeIn">
    <portfolio-widget-row />
    <bootstrap-table
      ref="table"
      :columns="columns"
      :data="data"
      :options="options">
    </bootstrap-table>
  </div>
</template>

<script>
import Vue from 'vue'
import api from "../../../shared/api";
import PortfolioWidgetRow from "../../dashboard/PortfolioWidgetRow";
import SeverityProgressBar from "../../components/SeverityProgressBar";

export default {
  components: {
    PortfolioWidgetRow
  },
  data() {
    return {
      columns: [
        {
          title: this.$t('message.project_name'),
          field: "name",
          sortable: true
        },
        {
          title: this.$t('message.version'),
          field: "version",
          sortable: true
        },
        {
          title: this.$t('message.last_bom_import'),
          field: "lastBomImport",
          sortable: true,
          formatter(timestamp, row, index) {
            return typeof timestamp === "number"
              ? new Date(timestamp).toDateString()
              : "-";
          }
        },
        {
          title: this.$t('message.bom_format'),
          field: "lastBomImportFormat",
          sortable: true
        },
        {
          title: this.$t('message.risk_score'),
          field: "metrics.inheritedRiskScore",
          sortable: true
        },
        {
          title: this.$t('message.active'),
          field: "active",
          formatter(value, row, index) {
            return value === true ? '<i class="fa fa-check-square-o" />' : "";
          },
          align: "center",
          sortable: true
        },
        {
          title: this.$t('message.vulnerabilities'),
          field: "metrics",
          sortable: true,
          formatter(metrics, row, index) {
            if (typeof metrics === "undefined") {
              return "-"; // No vulnerability info available
            }

            // Programmatically instantiate SeverityProgressBar Vue component
            let ComponentClass = Vue.extend(SeverityProgressBar);
            let progressBar = new ComponentClass({
              propsData: {
                vulnerabilities: metrics.vulnerabilities,
                critical: metrics.critical,
                high: metrics.high,
                medium: metrics.medium,
                low: metrics.low,
                unassigned: metrics.unassigned }
            });
            progressBar.$mount();
            return progressBar.$el.outerHTML;
          }
        }
      ],
      data: [],
      options: {
        search: true,
        showColumns: true,
        showRefresh: true,
        pagination: true,
        silentSort: false,
        sidePagination: 'server',
        queryParamsType: 'pageSize',
        pageList: '[10, 25, 50, 100]',
        pageSize: 10,
        icons: {
          refresh: 'fa-refresh'
        },
        responseHandler: function (res, xhr) {
          res.total = xhr.getResponseHeader("X-Total-Count");
          return res;
        },
        url: `${api.BASE_URL}/${api.URL_PROJECT}`
      }
    };
  }
};
</script>
