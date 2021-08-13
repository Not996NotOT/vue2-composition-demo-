<script>
import { h, ref, defineComponent, onMounted } from "@vue/composition-api";
import style from "./Common.module.scss";
import { List } from "linqts";

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
          on-input={(value) => {
            elInputValue.value = value;
          }}
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

export const useElTable = ({
  data: elData = [],
  columns = [],
  actionsColumns = [],
  loading: l = true,
} = {}) => {
  const data = ref(elData);
  const elActionsColumns = actionsColumns;
  const elColumns = columns;
  const loading = ref(l);
  const onInitialElTableData = (event) => {
    loading.value = true;
    event({ data });
    loading.value = false;
  };
  const setTableData = ({ list }) => {
    data.value = list;
    loading.value = false;
  };
  const template = defineComponent({
    setup() {
      return () => {
        return (
          <el-table data={data.value} v-loading={loading.value} width="100%">
            {elColumns.map((item) => {
              return item.render ? (
                item.render()
              ) : (
                <el-table-column
                  align="center"
                  key={item.prop}
                  prop={item.prop}
                  label={item.label}
                ></el-table-column>
              );
            })}
            {elActionsColumns.map((item) => item.render && item.render())}
          </el-table>
        );
      };
    },
  });
  return {
    loading,
    data,
    setTableData,
    onInitialElTableData,
    template,
  };
};

export const useElDialog = ({ width = "80%", title = "" } = {}) => {
  const elDialogVisible = ref(false);
  const toggleElDialogVisible = () =>
    (elDialogVisible.value = !elDialogVisible.value);
  const template = defineComponent({
    setup(props, ctx) {
      return () =>
        h("div", {}, [
          <el-dialog
            title={title}
            on={{
              ["update:visible"]: () => toggleElDialogVisible(),
            }}
            visible={elDialogVisible.value}
            width={width}
          >
            {ctx.slots.default && ctx.slots.default()}
          </el-dialog>,
        ]);
    },
  });
  return {
    toggleElDialogVisible,
    template,
  };
};

const useElCard = () => {
  const template = defineComponent({
    props: {
      title: String,
    },
    setup(props, ctx) {
      return () => (
        <el-card class={style.myElCard}>
          <template slot="header">
            <div class={style.myElCardTitle}>
              <div class={style.myElCardLeft}>
                {ctx.slots.title ? ctx.slots.title() : props.title}
              </div>
              <div class={style.myElCardActions}>
                {ctx.slots.actions && ctx.slots.actions()}
              </div>
            </div>
          </template>
          {ctx.slots.default && ctx.slots.default()}
        </el-card>
      );
    },
  });
  return {
    template,
  };
};

const useElPagination = ({ onCurrentPageChange: cChange = () => {} } = {}) => {
  const total = ref(0);
  const currentPage = ref(1);
  const template = defineComponent({
    setup() {
      return () => (
        <el-pagination
          background
          layout="prev, pager, next"
          total={total.value}
          on-current-change={currentChange}
          currentPage={currentPage.value}
          on={{
            ["update:currentPage"]: (value) => {
              currentPage.value = value;
            },
          }}
        ></el-pagination>
      );
    },
  });
  const setTotal = ({ total: t = 1 } = {}) => {
    total.value = t;
  };
  const getCurrentPage = () => {
    return currentPage.value;
  };
  const currentChange = () => {
    cChange(currentPage.value);
  };
  return {
    setTotal,
    getCurrentPage,
    template,
  };
};

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
      class={style.myElSelect}
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
  const getConditions = () => {
    let results = {};
    if (components.length > 0) {
      components.forEach((item) => {
        results[item.key] = item.component.getValue();
      });
    }
    return results;
  };
  const onButtonClick = () => {
    btnClick({ results: getConditions() });
  };
  const searchButton = useElButton({
    onClick: onButtonClick,
    text: buttonText,
  });
  const template = (
    <el-form inline={true} class={style.mySearch}>
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
                <div>{item.key}</div>
              </el-form-item>
            );
          }
        })}
      <searchButton.template />
    </el-form>
  );
  return {
    getConditions,
    template: defineComponent({
      setup() {
        return () => template;
      },
    }),
  };
};

const useMyTable = ({
  columns = [],
  currentPage: cPage = 1,
  pageSize = 10,
  onCurrentPageChange: cPageChange = () => {},
} = {}) => {
  let elCurrentPage = cPage;
  const table = useElTable({
    columns: columns,
  });
  const pagination = useElPagination({
    onCurrentPageChange: (currentPage) => {
      elCurrentPage = currentPage;
      cPageChange(currentPage);
    },
  });
  const initialData = (event) => {
    table.loading.value = true;
    event({
      currentPage: elCurrentPage,
      pageSize,
      loading: table.loading,
      setTableData: ({ list, total }) => {
        table.setTableData({ list });
        pagination.setTotal({ total });
      },
    });
  };

  const template = () => (
    <div class={style.myTable}>
      <table.template />
      <pagination.template />
    </div>
  );

  return {
    initialData,
    template: defineComponent({
      setup() {
        return () => h("div", {}, [template()]);
      },
    }),
  };
};

const MyFormItemType = {
  Input: Symbol("Input"),
  Select: Symbol("Select"),
  TextArea: Symbol("TextArea"),
};

const useMyForm = ({
  formItems = [],
  labelWidth = "80px",
  okButtonText = "确定",
  okButtonClick = () => {},
  cancelButtonText = "取消",
  cancelButtonClick = () => {},
} = {}) => {
  const components = [];
  const getFormData = () => {
    let results = {};
    if (components.length > 0) {
      components.forEach((item) => {
        results[item.key] = item.component.getValue();
      });
    }
    return results;
  };
  const okButton = useElButton({
    type: ElButtonType.Primary,
    text: okButtonText,
    onClick: () => {
      okButtonClick({ formData: getFormData() });
    },
  });
  const cancelButton = useElButton({
    type: ElButtonType.Info,
    text: cancelButtonText,
    onClick: cancelButtonClick,
  });
  const template = (
    <el-form label-width={labelWidth}>
      {formItems &&
        formItems.map((item) => {
          if (item.type == MyFormItemType.Input) {
            const input = useElInput({ placeholder: item.placeholder });
            components.push({ key: item.key, component: input });
            return (
              <el-form-item label={item.label}>
                <input.template />
              </el-form-item>
            );
          } else if (item.type == MyFormItemType.Select) {
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
                <div>{item.key}</div>
              </el-form-item>
            );
          }
        })}
      <el-form-item>
        <okButton.template />
        <cancelButton.template />
      </el-form-item>
    </el-form>
  );
  return {
    getFormData,
    template: defineComponent({
      setup() {
        return () => template;
      },
    }),
  };
};

const personList = new List([]);
class PersonService {
  constructor() {
    for (let i = 0; i < 100; i++) {
      personList.Add({
        name: `name${i}`,
        sex: i % 2 == 0 ? "女" : "男",
        age: i.toString(),
        address: `地址${i}`,
      });
    }
  }
  async getList({ conditions = {}, currentPage = 1, pageSize = 10 } = {}) {
    return new Promise((resovle) => {
      setTimeout(() => {
        let list = personList;
        Object.getOwnPropertyNames(conditions).forEach((item) => {
          if (conditions[item] !== "") {
            list = list.Where(
              (condition) => condition[item].indexOf(conditions[item]) !== -1
            );
            console.log(list);
          }
        });
        let pagelist = list.Skip((currentPage - 1) * pageSize).Take(pageSize);
        resovle({
          list: pagelist.ToArray(),
          total: list.Count(),
        });
      }, 1000);
    });
  }
}

export default defineComponent({
  setup() {
    const dialog = useElDialog({ title: "表单", width: "50%" });
    const personService = new PersonService();
    const card = useElCard();
    const myTable = useMyTable({
      columns: [
        { label: "姓名", prop: "name" },
        { label: "年龄", prop: "age" },
        { label: "性别", prop: "sex" },
        { label: "住址", prop: "address" },
      ],
      onCurrentPageChange: () => {
        loadingData();
      },
    });
    const actionButton = useElButton({
      text: "添加",
      onClick: () => {
        dialog.toggleElDialogVisible();
      },
    });
    const loadingData = () => {
      myTable.initialData(async ({ currentPage, pageSize, setTableData }) => {
        let { list, total } = await personService.getList({
          conditions: mySearch.getConditions(),
          currentPage,
          pageSize,
        });
        setTableData({ list, total });
      });
    };
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
          type: MySearchType.Input,
        },
        {
          key: "sex",
          label: "性别",
          placeholder: "请输入性别",
          type: MySearchType.Select,
          data: [
            {
              value: "",
              label: "全部",
            },
            {
              value: "男",
              label: "男",
            },
            {
              value: "女",
              label: "女",
            },
          ],
        },
        {
          key: "address",
          label: "地址",
          placeholder: "请输入地址",
          type: MySearchType.Input,
        },
      ],
      onButtonClick: () => {
        loadingData();
      },
    });
    const myForm = useMyForm({
      formItems: [
        {
          key: "name",
          label: "姓名",
          placeholder: "请输入姓名",
          type: MyFormItemType.Input,
        },
        {
          key: "age",
          label: "年龄",
          placeholder: "请输入年龄",
          type: MyFormItemType.Input,
        },
        {
          key: "sex",
          label: "性别",
          placeholder: "请输入性别",
          type: MyFormItemType.Select,
          data: [
            {
              value: "",
              label: "全部",
            },
            {
              value: "男",
              label: "男",
            },
            {
              value: "女",
              label: "女",
            },
          ],
        },
        {
          key: "address",
          label: "地址",
          placeholder: "请输入地址",
          type: MyFormItemType.Input,
        },
      ],
      okButtonClick: ({ formData }) => {
        console.log(formData);
      },
      cancelButtonClick: () => {
        dialog.toggleElDialogVisible();
      },
    });
    onMounted(() => {
      console.log(useMyForm);
      console.log(myForm);
      loadingData();
    });
    return () => (
      <div>
        <card.template>
          <template slot="title">
            <mySearch.template />
          </template>
          <template slot="actions">
            <actionButton.template />
          </template>
          <myTable.template />
        </card.template>
        <dialog.template>
          <myForm.template />
        </dialog.template>
      </div>
    );
  },
});
</script>
