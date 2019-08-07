<template>
  <div class="multiTableComp">
    <div class="multiTableComp_title">{{title}}</div>
    <table class="multiTableComp_table">
      <thead>
      <tr>
        <th
            class="multiTableComp_theadTh"
            v-for="(head,index) in headings"
            :key="index"
            :style="column_styles[index]">
          {{head}}
        </th>
      </tr>
      </thead>
      <tbody class="multiTableComp_tbody">
      <template v-for="(value,key) in tables">
        <tr>
          <td colspan="headings.length" v-on:click="key_clicked(value)">
            <span :class="value.open ? 'multiTableComp_arrowIcon__open' : 'multiTableComp_arrowIcon__closed'">{{key}}</span>
          </td>
        </tr>
        <tr v-for="(datarow,row_idx) in value.data" v-show="value.open">
          <td
              v-for="(datacol,col_idx) in datarow"
              :style="column_styles[col_idx]"
              :class="compute_td_class(row_idx,col_idx,value.data.length)"
              v-on:click="cell_clicked(key,row_idx,col_idx)">{{datacol}}</td>
        </tr>
      </template>
      </tbody>
    </table>
  </div>
</template>

<script>
  export default {
    name: 'MultiTableComp',
    data() {
      return {
        tree_line_classes: null
      };
    },
    props: {
      title: {
        type: String,
        default: null
      },
      headings: {
        type: Array,
        required: true,
        default: () =>{
          return null;
        }
      },
      column_widths: {
        type: Array,
        default: ()=> {
          return null;
        }
      },
      tree_line: {
        type: String,
        default: 'black'
      },
      tables: {
        type: Object,
        default: null
      },
      css_variables: {
        type: Object,
        default: () => {
          return null;
        }
      }
    },
    computed: {
      column_styles: function(){
        let col_styles = [];
        if(this.headings !== null){
          const n_cells = this.headings.length;
          col_styles = new Array(n_cells);
          for(let i=0; i<n_cells; i++){
            let width_style = null;
            if(this.column_widths === null){
              width_style = 'width: 7.5rem';
            }else if(this.column_widths[i] === null){
              width_style = 'width: 7.5rem';
            }else {
              width_style = `width:${this.column_widths[i]}px;`;
            }
            col_styles[i] = width_style;
          }
        }
        return col_styles;
      }
    },
    methods: {
      compute_td_class: function(row_idx,col_idx,data_length){
        if(this.tree_line_classes !== null && col_idx === 0){
          if(row_idx < data_length-1){
            return this.tree_line_classes[0];
          }else {
            return this.tree_line_classes[1];
          }
        }else {
          return '';
        }
      },
      key_clicked: function(value){
        value.open = !value.open;
      },
      cell_clicked: function(table_key,data_row_idx,data_col_idx){
        this.$emit('multitable_comp_cell',{table_key:table_key,data_row_idx:data_row_idx,data_col_idx:data_col_idx});
      }
    },
    mounted() {
      if(this.tree_line === 'black') {
        this.tree_line_classes = ['multiTableComp_tbodyTd__startTreeSolidBlack','multiTableComp_tbodyTd__endTreeSolidBlack'];
      }else if(this.tree_line === 'white') {
        this.tree_line_classes = ['multiTableComp_tbodyTd__startTreeSolidWhite','multiTableComp_tbodyTd__endTreeSolidWhite'];
      }else if(this.tree_line === 'dotted'){
        this.tree_line_classes = ['multiTableComp_tbodyTd__startTreeDottedBlack','multiTableComp_tbodyTd__endTreeDottedBlack'];
      }else {
        this.tree_line_classes = ['multiTableComp_tbodyTd__startTreeSolidBlack','multiTableComp_tbodyTd__endTreeSolidBlack'];
      }
      if(this.css_variables !== null){
        for(let key of Object.keys(this.css_variables)){
          this.$el.style.setProperty(`--${key}`, this.css_variables[key]);
        }
      }
    }
  }
</script>

<style lang="less">
  :root {
    --multitable_comp_font_family: Verdana,serif;

    --multitable_comp_title_font_size: 1rem;
    --multitable_comp_title_color: black;

    --multitable_comp_thead_color: black;
    --multitable_comp_thead_border_bottom: 1px solid black;
    --multitable_comp_thead_background: transparent;

    --multitable_comp_tbody_height: 20rem;
    --multitable_comp_down_icon: '\21D3';
    --multitable_comp_cell_font_size: 1rem;
    --multitable_comp_cell_color: black;
  }

  .multiTableComp {
    display: flex;
    flex-direction: column;
    align-items: center;
    font-family: var(--multitable_comp_font_family);

    &_title {
      font-size: var(--multitable_comp_title_font_size);
      color: var(--multitable_comp_title_color);
      font-weight: bold;
      margin-bottom: .5rem;
    }

    &_table {
      table-layout: fixed;
      border-collapse: collapse;
    }

    &_theadTh{
      color: var(--multitable_comp_thead_color);
      border-bottom: var(--multitable_comp_thead_border_bottom);
      background: var(--multitable_comp_thead_background);
    }

    &_tbody {
      height: var(--multitable_comp_tbody_height);
      overflow-y: auto;
      overflow-x: hidden;
      &::-webkit-scrollbar-track {
        -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
        border-radius: 10px;
        background-color: #F5F5F5;
      }
      &::-webkit-scrollbar {
        height: 12px;
        width: 12px;
        background-color: transparent;
      }
      &::-webkit-scrollbar-thumb {
        border-radius: 10px;
        -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,.3);
        background-color: #D62929;
      }
    }

    &_arrowIcon__open::before {
      content: var(--multitable_comp_down_icon);
      display: inline-block;
      color: var(--multitable_comp_cell_color);
      transition: 600ms linear all;
      margin-right: .2rem;
      font-size: 1rem;
      cursor: pointer;
    }

    &_arrowIcon__closed::before {
      content: var(--multitable_comp_down_icon);
      display: inline-block;
      color: var(--multitable_comp_cell_color);
      transform: rotate(-90deg);
      transition: 600ms linear all;
      margin-right: .2rem;
      font-size: 1rem;
      cursor: pointer;
    }

    &_tbodyTd__startTreeSolidBlack {
      background: url(img/startTreeSolidBlack.png) 1rem 54% no-repeat;
      padding-left: 1.6rem;
    }
    &_tbodyTd__endTreeSolidBlack {
      background: url(img/endTreeSolidBlack.png) 1rem 54% no-repeat;
      padding-left: 1.6rem;
    }
    &_tbodyTd__startTreeSolidWhite {
      background: url(img/startTreeSolidWhite.png) 1rem 54% no-repeat;
      padding-left: 1.6rem;
    }
    &_tbodyTd__endTreeSolidWhite {
      background: url(img/endTreeSolidWhite.png) 1rem 54% no-repeat;
      padding-left: 1.6rem;
    }
    &_tbodyTd__startTreeDottedBlack {
      background: url(img/startTreeDottedBlack.png) 1rem 54% no-repeat;
      padding-left: 1.6rem;
    }
    &_tbodyTd__endTreeDottedBlack {
      background: url(img/endTreeDottedBlack.png) 1rem 54% no-repeat;
      padding-left: 1.6rem;
    }
  }
</style>