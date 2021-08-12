<script>
import { h, ref, defineComponent, onMounted } from "@vue/composition-api";
//import style from "./Common.module.scss";

const ElButtonType = {
  Text: "text",
  Primary: "primary",
  Danger: "danger",
  Info: "into",
};

export const useElButton = ({
  text = "",
  type = ElButtonType.Primary,
  onClick = () => {},
} = {}) => {
  const template = defineComponent({
    setup(props, ctx) {
      return () => (
        <el-button on-click={onClick} type={type}>
          {ctx.slots.default ? ctx.slots.default() : text}
        </el-button>
      );
    },
  });
  return {
    template,
  };
};

export const useElInput = ({ value = "", placeholder = "" } = {}) => {
  const elInputValue = ref(value);
  const elInputPlaceholder = placeholder;
  const template = defineComponent({
    setup() {
      return () => (
        <el-input
          placeholder={elInputPlaceholder}
          value={elInputValue.value}
          on-input={(value) => (elInputValue.value = value)}
        ></el-input>
      );
    },
  });
  const getValue = () => {
    return elInputValue.value;
  };
  return {
    getValue,
    template,
  };
};

// export const useElTable = ({
//   data = [],
//   columns = [],
//   actionsColumns = [],
//   loading = false,
// } = {}) => {
//   const elTableData = ref(data);
//   const elActionsColumns = actionsColumns;
//   const elColumns = columns;
//   const elTableLoading = ref(loading);
//   const onInitialElTableData = (event) => {
//     elTableLoading.value = true;
//     event({ elTableData });
//     elTableLoading.value = false;
//   };
//   const template = defineComponent({
//     setup() {
//       return () => (
//         <el-table
//           data={elTableData.value}
//           v-loading={elTableLoading.value}
//           width="100%"
//         >
//           {elColumns.map((item) => {
//             return item.render ? (
//               item.render()
//             ) : (
//               <el-table-column
//                 align="center"
//                 key={item.prop}
//                 prop={item.prop}
//                 label={item.label}
//               ></el-table-column>
//             );
//           })}
//           {elActionsColumns.map((item) => item.render && item.render())}
//         </el-table>
//       );
//     },
//   });
//   return {
//     elTableLoading,
//     elTableData,
//     elColumns,
//     onInitialElTableData,
//     template,
//   };
// };

// export const useElDialog = ({ width = "80%", title = "" } = {}) => {
//   const elDialogVisible = ref(false);
//   const toggleElDialogVisible = () =>
//     (elDialogVisible.value = !elDialogVisible.value);
//   const template = defineComponent({
//     setup(props, ctx) {
//       return () =>
//         h("div", {}, [
//           <el-dialog
//             title={title}
//             on={{
//               ["update:visible"]: () => toggleElDialogVisible(),
//             }}
//             visible={elDialogVisible.value}
//             width={width}
//           >
//             {ctx.slots.default && ctx.slots.default()}
//           </el-dialog>,
//         ]);
//     },
//   });
//   return {
//     toggleElDialogVisible,
//     template,
//   };
// };

// const useElCard = () => {
//   const template = defineComponent({
//     props: {
//       title: String,
//     },
//     setup(props, ctx) {
//       return () => (
//         <el-card class={style.myElCard}>
//           <template slot="header">
//             <div class={style.myElCardTitle}>
//               <div class={style.myElCardLeft}>
//                 {ctx.slots.title ? ctx.slots.title() : props.title}
//               </div>
//               <div class={style.myElCardActions}>
//                 {ctx.slots.actions && ctx.slots.actions()}
//               </div>
//             </div>
//           </template>
//           {ctx.slots.default && ctx.slots.default()}
//         </el-card>
//       );
//     },
//   });
//   return {
//     template,
//   };
// };

// const useElPagination = () => {
//   const total = ref(0);
//   const template = defineComponent({
//     setup() {
//       return () => (
//         <el-pagination
//           background
//           layout="prev, pager, next"
//           total={total.value}
//         ></el-pagination>
//       );
//     },
//   });
//   return {
//     template,
//   };
// };

const useElSelect = ({
  data: elSelectData = [],
  value: elSelectValue = "",
  placeholder = "请选择",
} = {}) => {
  const data = ref(elSelectData);
  const value = ref(elSelectValue);
  const getValue = () => value.value;
  const loading = ref(false);
  const onInitialData = (event) => {
    loading.value = true;
    event({ data, value });
    loading.value = false;
  };
  const returnTemplate = () => (
    <el-select
      value={value.value}
      on-input={(v) => (value.value = v)}
      placeholder={placeholder}
      loading={loading.value}
    >
      {data.value &&
        data.value.map((item) => {
          return (
            <el-option
              key={item.value}
              label={item.label}
              value={item.value}
            ></el-option>
          );
        })}
    </el-select>
  );
  const template = defineComponent({
    setup() {
      return () => h("div", {}, [returnTemplate()]);
    },
  });
  return {
    getValue,
    onInitialData,
    template,
  };
};

const MySearchType = {
  Input: Symbol("Input"),
  Select: Symbol("Select"),
};

const useMySearch = ({
  conditions = [],
  buttonText = "查询",
  onButtonClick: btnClick = () => {},
}) => {
  const components = [];
  const onButtonClick = () => {
    let results = {};
    if (components.length > 0) {
      components.forEach((item) => {
        results[item.key] = item.component.getValue();
      });
    }
    btnClick(results);
  };
  const searchButton = useElButton({
    onClick: onButtonClick,
    text: buttonText,
  });
  const template = (
    <el-form inline={true}>
      {conditions &&
        conditions.map((item) => {
          if (item.type == MySearchType.Input) {
            const input = useElInput({ placeholder: item.placeholder });
            components.push({ key: item.key, component: input });
            return (
              <el-form-item label={item.label}>
                <input.template />
              </el-form-item>
            );
          } else if (item.type == MySearchType.Select) {
            const select = useElSelect({
              placeholder: item.placeholder,
              data: item.data,
            });
            components.push({ key: item.key, component: select });
            return (
              <el-form-item label={item.label}>
                <select.template />
              </el-form-item>
            );
          } else {
            return (
              <el-form-item>
                <searchButton.template />
              </el-form-item>
            );
          }
        })}
      <searchButton.template />
    </el-form>
  );
  return {
    template: defineComponent({
      setup() {
        return () => template;
      },
    }),
  };
};

export default defineComponent({
  setup() {
    const mySearch = useMySearch({
      conditions: [
        {
          key: "name",
          label: "姓名",
          placeholder: "请输入姓名",
          type: MySearchType.Input,
        },
        {
          key: "age",
          label: "年龄",
          placeholder: "请输入年龄",
          type: MySearchType.Select,
          data: [
            {
              value: "",
              label: "全部",
            },
            {
              value: "1",
              label: "男",
            },
            {
              value: "0",
              label: "女",
            },
          ],
        },
        {
          key: "sex",
          label: "性别",
          placeholder: "请输入性别",
          type: MySearchType.Input,
        },
        {
          key: "address",
          label: "地址",
          placeholder: "请输入地址",
          type: MySearchType.Input,
        },
      ],
      onButtonClick: (results) => {
        console.log(results);
      },
    });
    onMounted(() => {
      console.log(123);
    });
    return () => (
      <div>
        <mySearch.template />
      </div>
    );
  },
});
</script>
