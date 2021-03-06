<script setup lang="ts">
import zhCN from 'ant-design-vue/es/locale/zh_CN';
import { h, reactive, ref, resolveComponent } from 'vue';
import arr2tree from '@any86/array-to-tree';
import Curd, { defineC, defineD, defineR, defineU } from '@/lib';
import axios from 'axios';
interface KV<T = any> {
  [k: string | number]: T;
}
type Row = {
  userId: string;
  userName: string;
  age: number;
  sex: 1 | 2;
  realName: string;
  department: string;
  avatar: string;
  state: 0 | 1 | 2;
  joinDate: string;
  email: string;
  position: string;
  role: string;
};

/**
 * 获取职位列表
 * @returns promise 列表
 */
function usePositionOptions() {
  const optionsRef = ref<{ label: string; value: string }[]>();

  async function get() {
    if (!optionsRef.value) {
      const { data } = await http.get<{ id: string; name: string }[]>('/global/position');
      optionsRef.value = data.map(({ id, name }) => {
        return { label: name, value: id };
      });
    }
    return optionsRef;
  }

  return [optionsRef, get] as const;
}

/**
 * 获取部门列表
 * @returns promise 列表
 */
function useDepartmentOptions() {
  const optionsRef = ref<{ label: string; value: string }[]>();

  async function get() {
    if (!optionsRef.value) {
      const { data } = await http.get<{ id: string; name: string }[]>('/global/department');
      optionsRef.value = data.map(({ id, name }) => {
        return { label: name, value: id };
      });
    }
    return optionsRef;
  }

  return [optionsRef, get] as const;
}

/**
 * 获取角色列表
 * @returns promise 列表
 */
function useRoleOptions() {
  const optionsRef = ref<{ label: string; value: string }[]>();

  async function get() {
    if (!optionsRef.value) {
      const { data } = await http.get<{ id: string; name: string }[]>('/global/role');
      optionsRef.value = data.map(({ id, name }) => {
        return { label: name, value: id };
      });
    }
    return optionsRef;
  }

  return [optionsRef, get] as const;
}

const http = axios.create({
  // 配置
  baseURL: 'https://mock.apifox.cn/m1/486559-0-default/api/',
});

const primaryKey = 'userId';
const [roleOptions, getRoleOptions] = useRoleOptions();
const [departmentOptions, getDepartmentOptions] = useDepartmentOptions();
const [positionOptions, getPositionOptions] = usePositionOptions();

const r = defineR({
  async before() {
    let treeData = reactive<KV[]>([]);
    const { data } = await http.get('/global/menu');
    treeData = arr2tree(data, {
      transform(node: any) {
        node.title = node.name;
        node.key = node.id;
        return node;
      },
    });
    return { treeData };
  },
  columns: [
    {
      title: '姓名',
      dataIndex: 'realName',
    },
    {
      title: '账户',
      dataIndex: 'userName',
    },
    {
      title: '部门',
      dataIndex: 'department',
    },
    {
      title: '性别',
      customRender({ record }: { record: Row }) {
        const { sex } = record;
        const map = {
          1: ['blue', '男'],
          2: ['pink', '女'],
        } as const;

        const color = map[sex][0];
        const text = map[sex][1];
        // 插槽的内容用函数性能更优
        return h(resolveComponent('a-tag'), { color }, () => text);
      },
    },

    {
      title: '年龄',
      dataIndex: 'age',
    },

    {
      title: '角色',
      dataIndex: 'role',
    },

    {
      title: '岗位',
      dataIndex: 'position',
    },
    {
      title: '邮箱',
      dataIndex: 'email',
    },

    {
      title: '入职日期',
      dataIndex: 'joinDate',
    },
    {
      title: '状态',
      customRender({ record }: { record: Row }) {
        const { state } = record;
        const map = {
          0: ['#999', '锁定'],
          1: ['#009688', '正常'],
          2: ['#f00', '离职'],
        };
        const type = map[state][0];
        const text = map[state][1];
        return h(resolveComponent('a-badge'), { color: type, text });
      },
    },
    {
      title: '操作',
      key: 'operation',
      fixed: 'right',
      width: 250,
    },
  ],
  conditionItems: (shared) => {
    return [{ is: 'a-input', name: 'name', label: '用户名' }];
  },
  async done(params) {
    const { data } = await http('/user', { params });
    return data;
  },
});
const c = defineC({
  async before(row) {
    await Promise.all([getRoleOptions(), getDepartmentOptions(), getPositionOptions()]);
  },
  async done(formData) {
    formData.state = formData.state ? 1 : 0;
    const { data } = await http.post('/user', formData);
    return data.msg;
  },

  items: () => [
    { is: 'a-input', name: 'userName', label: '账号' },
    { is: 'a-input', name: 'realName', label: '姓名' },
    {
      is: 'a-radio-group',
      name: 'sex',
      label: '性别',
      props: {
        options: [
          { label: '男', value: 1 },
          { label: '女', value: 2 },
        ],
      },
    },
    { is: 'a-input-number', name: 'age', label: '年龄' },
    { is: 'a-input', name: 'email', label: '邮箱' },
    { is: 'a-select', name: 'departmentId', label: '部门', props: { options: departmentOptions } },
    { is: 'a-select', name: 'roleId', label: '角色', props: { options: roleOptions } },
    { is: 'a-select', name: 'positonId', label: '职位', props: { options: positionOptions } },
    {
      is: 'a-radio-group',
      name: 'state',
      label: '状态',
      props: {
        options: [
          { label: '正常', value: 1 },
          { label: '离职', value: 2 },
          { label: '锁定', value: 0 },
        ],
      },
    },
    {
      is: 'a-tree',
      name: 'menuId',
      modelName: 'checkedKeys',
      label: '菜单',
      props: {
        // treeData,
        checkable: true,
        defaultExpandAll: true,
        // selectable:true,
        // autoExpandParent:true,
      },
    },
  ],
});

const u = defineU({
  async before(row) {
    const { data } = await http.get('/user/' + row[primaryKey]);
    data.state = Boolean(data.state);
    await Promise.all([getRoleOptions(), getDepartmentOptions(), getPositionOptions()]);
    return data;
  },

  async done(formData) {
    const { id, ...kv } = formData;
    const { data } = await http.put('/user/' + id, kv);
    return data.msg;
  },

  items: c.items,
});

const d = defineD({
  async done(idList) {
    if (1 < idList.length) {
      const { data } = await http.delete('/user/', {
        params: {
          idList,
        },
      });
      return data.msg;
    } else {
      const { data } = await http.delete('/user/' + idList[0]);
      return data.msg;
    }
  },
});
</script>

<template>
  <a-config-provider :locale="zhCN">
    <curd v-bind="{ primaryKey, c, u, r, d }"></curd>
  </a-config-provider>
</template>
