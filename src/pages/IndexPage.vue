<!-- eslint-disable max-len -->
<template>
  <div class="q-pa-md">
    <q-form @submit="handleFilter(form.query)" class="flex q-mb-md">
      <q-input filled v-model="form.query" label="Digite um cep ou rua" style="width: 300px" />
      <q-btn label="Buscar" type="submit" color="primary" class="q-ml-md" style="height: 54px;" />
    </q-form>
    <q-table title="Users" :rows="enderecos" :columns="columns" row-key="name" no-data-label="Nenhum endereço cadastrado">
      <template v-slot:top>
        <q-space />
        <q-btn color="primary" label="Novo endereço" :to="{ name: 'formEndereco' }" />
      </template>
      <template v-slot:body-cell-actions="props">
        <q-td :props="props" class="q-gutter-sm">
          <q-btn icon="edit" color="info" dense size="sm" @click="handleEditUser(props.row.id)" />
          <q-btn icon="delete" color="negative" dense size="sm" @click="handleDeleteUser(props.row.id)" />
        </q-td>
      </template>
    </q-table>
  </div>
</template>

<script>
import { defineComponent, ref, onMounted } from 'vue';
import { api } from 'boot/axios';
import { useQuasar } from 'quasar';
import { useRouter } from 'vue-router';
import axios from "axios";

export default defineComponent({
  name: 'IndexPage',
  setup() {
    const enderecos = ref([]);

    const form = ref({
      query: '',
    });

    const columns = [
      {
        name: 'rua', field: 'rua', label: 'Rua', sortable: true, align: 'left',
      }, {
        name: 'numero', field: 'numero', label: 'Número', sortable: true, align: 'left',
      }, {
        name: 'bairro', field: 'bairro', label: 'Bairro', sortable: true, align: 'left',
      }, {
        name: 'cep', field: 'cep', label: 'CEP', sortable: true, align: 'left',
      }, {
        name: 'actions', field: 'actions', label: 'Ações', align: 'right',
      },
    ];

    const $q = useQuasar();
    const router = useRouter();

    const getUsers = async () => {
      try {
        const { data } = await api.get('endereco');
        enderecos.value = data;
      } catch (error) {
        throw new Error(error);
      }
    };
    // eslint-disable-next-line consistent-return
    const remove = async (id) => {
      try {
        const data = await api.delete(`endereco/${id}`);
        return data;
      } catch (error) {
        throw new Error(error);
      }
    };

    const handleEditUser = async (id) => {
      router.push({ name: 'formEndereco', params: { id } });
    };

    const handleFilter = async (query) => {
      try {
        const { data } = await api.get(`endereco/buscar/${query}`);
        enderecos.value = data;
      } catch (error) {
        $q.notify({ message: 'Usuário não encontrado', icon: 'times', color: 'negative' });
      }
    };

    const handleDeleteUser = async (id) => {
      try {
        $q.dialog({
          dark: false,
          title: 'Confirmação de deleção',
          message: `Deseja realmente excluir este endereço ?`,
          cancel: true,
          persistent: true,
        }).onOk(async () => {
          await remove(id);
          $q.notify({ message: 'Endereço deletado', icon: 'check', color: 'positive' });
          await getUsers();
        });
      } catch (error) {
        $q.notify({ message: 'Error User delete', icon: 'times', color: 'negative' });
      }
    };

    onMounted(() => {
      getUsers();
    });

    return {
      columns,
      enderecos,
      form,
      handleDeleteUser,
      handleEditUser,
      handleFilter,
    };
  },
});

</script>
