<script>
import { h, ref, defineComponent, onMounted } from "@vue/composition-api";
import style from "./Common.module.scss";
import { List } from "linqts";
import { Message, MessageBox, Notification } from "element-ui";
import dayjs from "dayjs";

const MyMessageEnum = {
  Error: "error",
  Warning: "warning",
  Success: "success",
};

class MyMessage {
  /**
   * 弹出消息
   * @param {string} message 消息
   * @param {MyMessageEnum} MyMessageEnum
   */
  static showMessage(message, myMessageEnum = MyMessageEnum.Error) {
    Message({ message, type: myMessageEnum });
  }

  /**
   * 确认框
   * @param {string} message 消息
   * @param {string} okEvent 成功回调事件
   * @param {MyMessageEnum} myMessageEnum
   * @param {string} title 标题
   * @param {string} confirmButtonText 确定显示文字
   * @param {string} cancelButtonText 取消显示文字
   */
  static confirm(
    message,
    okEvent,
    myMessageEnum = MyMessageEnum.Warning,
    title = "提示",
    confirmButtonText = "确定",
    cancelButtonText = "取消"
  ) {
    MessageBox.confirm(message, title, {
      confirmButtonText,
      cancelButtonText,
      type: myMessageEnum,
    })
      .then(okEvent)
      .catch(() => {});
  }

  static notify(message, title = "提示", duration = "0") {
    Notification({
      title,
      message,
      duration,
    });
  }
}

const ElDatePickerType = {
  Date: "date",
};

export const useElDatePicker = ({
  placeholder: ph = "选择日期",
  type = ElDatePickerType.Date,
  value: paramV = dayjs().format("YYYY-MM-DD"),
} = {}) => {
  const value = ref(paramV);
  const getValue = () => value.value;
  const setValue = (v) =>
    (value.value = v == "" ? dayjs().format("YYYY-MM-DD") : v);
  const returnTemplate = () => {
    return (
      <div>
        <el-date-picker
          class={style.myElDatePicker}
          value={value.value}
          on-input={(v) => {
            value.value = v;
          }}
          value-format="yyyy-MM-dd"
          type={type}
          placeholder={ph}
        ></el-date-picker>
      </div>
    );
  };
  return {
    setValue,
    getValue,
    template: defineComponent({
      setup() {
        return () => h("div", {}, [returnTemplate()]);
      },
    }),
  };
};

const useElDatePickerRange = ({
  rangeSeparator = "至",
  startPlaceholder = "开始日期",
  endPlaceholder = "结束日期",
  value: v = new Date().toUTCString(),
} = {}) => {
  console.log(dayjs());
  const value = ref(v);
  const getValue = () => value.value;
  const setValue = (v = value.value = v);
  return {
    setValue,
    getValue,
    template: defineComponent({
      setup() {
        return (
          <el-date-picker
            v-model={value}
            type="daterange"
            range-separator={rangeSeparator}
            start-placeholder={startPlaceholder}
            end-placeholder={endPlaceholder}
          ></el-date-picker>
        );
      },
    }),
  };
};

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
    props: ["click"],
    setup(props, ctx) {
      return () => (
        <el-button on-click={props.click ? props.click : onClick} type={type}>
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
  const isRequire = ref(false);
  const template = defineComponent({
    props: ["validators"],
    setup(props) {
      let validators = props.validators;
      const validatorInput = () => {
        if (validators && validators.required) {
          isRequire.value = !elInputValue.value;
        }
      };
      return () => (
        <div class={style.formContainer}>
          <el-input
            placeholder={elInputPlaceholder}
            value={elInputValue.value}
            on-input={(value) => {
              elInputValue.value = value;
              validatorInput();
            }}
            on-blur={() => validatorInput()}
            class={isRequire.value ? style.formValidator : ""}
          ></el-input>
          <div class={style.formErrorMessage}>
            {isRequire.value && validators.message}
          </div>
        </div>
      );
    },
  });
  const getValue = () => {
    return elInputValue.value;
  };
  const setValue = (value) => {
    elInputValue.value = value;
  };
  const resetValidator = () => (isRequire.value = false);
  return {
    resetValidator,
    getValue,
    setValue,
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
  console.log("tableColumns", columns);
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
              console.log(item.render);
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
            {elActionsColumns.map((item) => {
              return item.render ? (
                item.render()
              ) : (
                <el-table-column
                  align="center"
                  key={item.prop}
                  prop={item.prop}
                  label={item.label}
                  {...{
                    scopedSlots: {
                      default: (scope) => {
                        return item.scope({ scope });
                      },
                    },
                  }}
                ></el-table-column>
              );
            })}
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
        <div class={style.elPagination}>
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
        </div>
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
  const setValue = (v) => (value.value = v);
  const loading = ref(false);
  const isRequire = ref(false);
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
  const resetValidator = () => (isRequire.value = false);
  const template = defineComponent({
    setup() {
      return () => h("div", {}, [returnTemplate()]);
    },
  });
  return {
    resetValidator,
    setValue,
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
  actionsColumns = [],
} = {}) => {
  let elCurrentPage = cPage;
  const table = useElTable({
    columns,
    actionsColumns,
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
  const toggleTableLoading = () => (table.loading.value = !table.loading.value);

  const template = () => (
    <div class={style.myTable}>
      <table.template />
      <pagination.template />
    </div>
  );

  return {
    toggleTableLoading,
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
  DatePicker: Symbol("DatePicker"),
  DatePickerRange: Symbol("DatePickerRange"),
};

const useMyForm = ({
  formItems = [],
  labelWidth = "80px",
  splitColumnsCount = 2,
  okButtonText = "确定",
  okButtonClick = () => {},
  cancelButtonText = "取消",
  cancelButtonClick = () => {},
} = {}) => {
  let formId = undefined;
  const components = [];
  const getFormData = () => {
    let results = {};
    if (components.length > 0) {
      components.forEach((item) => {
        results[item.key] = item.component.getValue();
      });
    }
    if (formId !== undefined) {
      results.id = formId;
    }
    return results;
  };
  const initialFormData = ({ id, data }) => {
    formId = id;
    console.log(formId);
    if (data) {
      components.forEach((item) => {
        item.component.setValue(data[item.key]);
      });
    }
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
  const clearFormData = () => {
    components.forEach((item) => {
      item.component.setValue("");
      if (item.component.resetValidator) {
        item.component.resetValidator();
      }
    });
  };
  const template = (
    <div class={style.myForm}>
      <el-form label-width={labelWidth}>
        {formItems &&
          formItems.map((item) => {
            if (item.type == MyFormItemType.Input) {
              const input = useElInput({ placeholder: item.placeholder });
              components.push({ key: item.key, component: input });
              return (
                <el-form-item
                  label={item.label}
                  style={{ width: `${100 / splitColumnsCount}%` }}
                  rules={item.validators}
                >
                  <input.template validators={item.validators} />
                </el-form-item>
              );
            } else if (item.type == MyFormItemType.Select) {
              const select = useElSelect({
                placeholder: item.placeholder,
                data: item.data,
              });
              components.push({ key: item.key, component: select });
              return (
                <el-form-item
                  label={item.label}
                  style={{ width: `${100 / splitColumnsCount}%` }}
                  rules={item.validators}
                >
                  <select.template />
                </el-form-item>
              );
            } else if (item.type == MyFormItemType.DatePicker) {
              const datePicker = useElDatePicker({
                placeholder: item.placeholder,
              });
              components.push({ key: item.key, component: datePicker });
              return (
                <el-form-item
                  label={item.label}
                  style={{ width: `${100 / splitColumnsCount}%` }}
                >
                  <datePicker.template />
                </el-form-item>
              );
            } else if (item.type == MyFormItemType.DatePickerRange) {
              const datePickerRange = useElDatePickerRange({
                placeholder: item.placeholder,
              });
              return (
                <el-form-item
                  label={item.label}
                  style={{ width: `${100 / splitColumnsCount}%` }}
                  rules={item.validators}
                >
                  <datePickerRange.template />
                </el-form-item>
              );
            } else {
              return (
                <div style={{ display: "none" }}>
                  <div>{item.key}</div>
                </div>
              );
            }
          })}
        <div class={style.myFormButtons}>
          <okButton.template />
          <cancelButton.template />
        </div>
      </el-form>
    </div>
  );
  return {
    clearFormData,
    initialFormData,
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
        id: i,
        name: `name${i}`,
        sex: i % 2 == 0 ? "女" : "男",
        age: i.toString(),
        address: `地址${i}`,
        createtime: `2019-02-03`,
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
        let pagelist = list
          .OrderByDescending((item) => item.id)
          .Skip((currentPage - 1) * pageSize)
          .Take(pageSize);

        resovle({
          list: pagelist.ToArray(),
          total: list.Count(),
        });
      }, 1000);
    });
  }
  async removeById({ id }) {
    return new Promise((resovle) => {
      setTimeout(() => {
        let entity = personList.Where((item) => item.id == id).FirstOrDefault();
        resovle(personList.Remove(entity));
      }, 1000);
    });
  }
  async update({ data }) {
    console.log("update", data);
    return new Promise((resovle) => {
      setTimeout(() => {
        let entity = personList
          .Where((item) => item.id == data.id)
          .FirstOrDefault();
        if (data) {
          Object.getOwnPropertyNames(data).forEach((item) => {
            entity[item] = data[item];
          });
        }
        resovle(true);
      }, 1000);
    });
  }

  async add({ data }) {
    return new Promise((resovle) => {
      setTimeout(() => {
        data.id = Math.random();
        personList.Add(data);
        console.log(data);
        resovle(true);
      }, 1000);
    });
  }
}

export default defineComponent({
  setup() {
    const editButton = useElButton({
      type: ElButtonType.Info,
      text: "编辑",
      onClick: () => {
        console.log("edit");
      },
    });
    const deleteButton = useElButton({
      type: ElButtonType.Danger,
      text: "删除",
      onClick: () => {
        console.log("delete");
      },
    });
    const dialog = useElDialog({ title: "表单", width: "80%" });
    const personService = new PersonService();
    const card = useElCard();
    const myTable = useMyTable({
      columns: [
        {
          render: () => (
            <el-table-column
              align="center"
              key="id"
              prop="id"
              label="id"
            ></el-table-column>
          ),
        },
        { label: "姓名", prop: "name" },
        { label: "年龄", prop: "age" },
        { label: "性别", prop: "sex" },
        { label: "住址", prop: "address" },
        { label: "创建时间", prop: "createtime" },
      ],
      actionsColumns: [
        {
          scope: ({ scope }) => {
            return (
              <div>
                <editButton.template
                  click={() => {
                    myForm.initialFormData({
                      id: scope.row.id,
                      data: scope.row,
                    });
                    dialog.toggleElDialogVisible();
                  }}
                />
                <deleteButton.template
                  click={() => {
                    MyMessage.confirm("确认删除这条数据么", async () => {
                      myTable.toggleTableLoading();
                      if (
                        await personService.removeById({ id: scope.row.id })
                      ) {
                        myTable.toggleTableLoading();
                        loadingData();
                        MyMessage.showMessage(
                          "删除成功",
                          MyMessageEnum.Success
                        );
                      }
                    });
                  }}
                />
              </div>
            );
          },
        },
      ],
      onCurrentPageChange: () => {
        loadingData();
      },
    });
    const refreshButton = useElButton({
      type: ElButtonType.Info,
      text: "刷新",
      onClick: () => loadingData(),
    });
    const myForm = useMyForm({
      okButtonText: "保存",
      splitColumnsCount: 2,
      formItems: [
        {
          key: "id",
          label: "id",
          hidden: true,
        },
        {
          key: "name",
          label: "姓名",
          placeholder: "请输入姓名",
          type: MyFormItemType.Input,
          validators: {
            required: true,
            message: "请输入姓名",
          },
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
        {
          key: "createtime",
          label: "创建时间",
          type: MyFormItemType.DatePicker,
        },
      ],
      okButtonClick: ({ formData }) => {
        if (formData.id !== undefined) {
          MyMessage.confirm("确认更新这条数据么？", async () => {
            dialog.toggleElDialogVisible();
            myTable.toggleTableLoading();
            if (await personService.update({ data: formData })) {
              MyMessage.showMessage("更新成功", MyMessageEnum.Success);
              myTable.toggleTableLoading();
            }
          });
        } else {
          MyMessage.confirm("确认添加这条数据么", async () => {
            dialog.toggleElDialogVisible();
            myTable.toggleTableLoading();
            if (await personService.add({ data: formData })) {
              MyMessage.showMessage("添加成功", MyMessageEnum.Success);
              myTable.toggleTableLoading();
            }
          });
        }
      },
      cancelButtonClick: () => {
        dialog.toggleElDialogVisible();
      },
    });
    const actionButton = useElButton({
      text: "添加",
      onClick: () => {
        myForm.clearFormData();
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

    onMounted(() => {
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
            <refreshButton.template />
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
