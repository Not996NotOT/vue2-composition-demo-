<script>
import { ref, defineComponent, onMounted } from "@vue/composition-api";
import style from "./Common.module.scss";

const ElButtonType = {
  Text: "text",
  Primary: "primary",
  Danger: "danger",
  Info: "into",
};

export const useElButton = ({
  elButtonText = "",
  elButtonType = ElButtonType.Primary,
  elButtonClick = () => {},
} = {}) => {
  const template = defineComponent({
    setup(props, ctx) {
      return () => (
        <el-button on-click={elButtonClick} type={elButtonType}>
          {ctx.slots.default ? ctx.slots.default() : elButtonText}
        </el-button>
      );
    },
  });
  return {
    template,
  };
};

export const useElInput = ({
  elInputValue: inputValue = "",
  elInputPlaceholder: placeholder = "",
} = {}) => {
  const elInputValue = ref(inputValue);
  const elInputPlaceholder = placeholder;
  const template = defineComponent({
    setup() {
      return () => (
        <el-input
          value={elInputValue.value}
          placeholder={elInputPlaceholder}
          on-input={(value) => (elInputValue.value = value)}
        ></el-input>
      );
    },
  });
  return {
    elInputValue,
    template,
  };
};

export const useElTable = ({
  tableData = [],
  columns = [],
  actionsColumns = [],
  tableLoading = false,
} = {}) => {
  const elTableData = ref(tableData);
  const elColumns = columns;
  const elTableLoading = ref(tableLoading);
  const onInitialElTableData = (event) => {
    elTableLoading.value = true;
    event({ elTableData });
    elTableLoading.value = false;
  };
  const template = defineComponent({
    setup() {
      return () => (
        <el-table
          data={elTableData.value}
          v-loading={elTableLoading.value}
          width="100%"
        >
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
          {actionsColumns.map((item) => item.render && item.render())}
        </el-table>
      );
    },
  });
  return {
    elTableLoading,
    elTableData,
    elColumns,
    onInitialElTableData,
    template,
  };
};

export const useElDialog = ({
  elDialogWitdh = "80%",
  elDialogTitle = "",
} = {}) => {
  const elDialogVisible = ref(false);
  const toggleElDialogVisible = () =>
    (elDialogVisible.value = !elDialogVisible.value);
  const template = defineComponent({
    setup(props, ctx) {
      return () => (
        <el-dialog
          title={elDialogTitle}
          on={{
            ["update:visible"]: () => toggleElDialogVisible(),
          }}
          visible={elDialogVisible.value}
          width={elDialogWitdh}
        >
          {ctx.slots.default && ctx.slots.default()}
        </el-dialog>
      );
    },
  });
  return {
    elDialogVisible,
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

const useElPagination = () => {
  const total = ref(0);
  const template = defineComponent({
    setup() {
      return () => (
        <el-pagination
          background
          layout="prev, pager, next"
          total={total.value}
        ></el-pagination>
      );
    },
  });
  return {
    template,
  };
};

export default {
  setup() {
    const card = useElCard();
    const pagination = useElPagination();
    const dialog = useElDialog();
    const input = useElInput({ elInputPlaceholder: "请输入查询条件" });
    const searchButton = useElButton({
      elButtonText: "查询",
      elButtonClick: () => {
        console.log(input.elInputValue.value);
      },
    });
    const addButton = useElButton({
      elButtonText: "添加",
      elButtonType: ElButtonType.Primary,
      elButtonClick: () => {
        console.log(123);
      },
    });
    const editButton = useElButton({
      elButtonText: "修改",
      elButtonType: ElButtonType.Info,
      elButtonClick: () => {
        console.log(123);
      },
    });
    const table = useElTable({
      columns: [
        { label: "todo item", prop: "item" },
        { label: "datetime", prop: "datetime" },
        { label: "isFinished", prop: "isFinished" },
      ],
      actionsColumns: [
        {
          render: () => (
            <el-table-column
              align="center"
              label="操作"
              formatter={() => (
                <div>
                  <editButton.template />
                  <el-button type="danger">删除</el-button>
                </div>
              )}
            ></el-table-column>
          ),
        },
      ],
    });
    onMounted(() => {
      table.onInitialElTableData(({ elTableData }) => {
        elTableData.value = [
          { item: "todo1", datetime: "2019-01-01", isFinished: false },
        ];
      });
    });
    return () => (
      <div>
        <el-row>
          <card.template>
            <template slot="title">请选择</template>
            <div class={style.myElSelect}>
              <el-button>亲亲</el-button>
            </div>
          </card.template>
        </el-row>
        <el-row>
          <card.template>
            <template slot="title">
              <input.template />
              <searchButton.template />
            </template>
            <template slot="actions">
              <addButton.template />
            </template>
            <div class={style.myElTable}>
              <table.template />
              <pagination.template />
            </div>
          </card.template>
        </el-row>
        <dialog.template />
      </div>
    );
  },
};
</script>
