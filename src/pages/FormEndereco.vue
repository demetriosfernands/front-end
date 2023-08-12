<!-- eslint-disable max-len -->
<template>
  <q-page padding>
    <q-form @submit="handleSubmit()" class="row q-col-gutter-sm">
      <q-input filled v-model="form.rua" label="Nome da rua" lazy-rules :rules="[val => val && val.length > 0 || 'Digite algum nome']" class="col-lg-6 col-xs-12" type="text" />
      <q-input filled v-model="form.numero" label="Número" lazy-rules class="col-lg-6 col-xs-12" type="text" :rules="[val => val && val.length > 0 || 'Digite o número']" />
      <q-input filled v-model="form.bairro" label="Bairro" lazy-rules :rules="[val => val && val.length > 0 || 'Nome do bairro']" class="col-lg-6 col-xs-12" type="text" />
      <q-input filled v-model="form.cep" @keyup="buscaPorCep()" label="CEP" lazy-rules :rules="[val => val && val.length > 0 || 'Somente números']" class="col-lg-6 col-xs-12" type="number" />
      <q-select v-model="form.id" class="col-lg-6 col-xs-12" style="display: none;" />

      <div class="col-12 q-gutter-sm">
        <q-btn label="Save" type="submit" color="primary" class="float-right" icon="save" @submit="handleSubmit()" />
        <q-btn label="Cancelar" color="white" class="float-right" text-color="primary" :to="{ name: 'home' }" />
      </div>
    </q-form>
  </q-page>
</template>

<script>
import { defineComponent, ref, onMounted } from 'vue';
import { api } from 'boot/axios';
import { useQuasar } from 'quasar';
import { useRouter, useRoute } from 'vue-router';
import axios from 'axios';

export default defineComponent({
  name: 'FormEndereco',
  setup() {
    const form = ref({
      id: '',
      rua: '',
      numero: '',
      bairro: '',
      cep: '',
    });

    const route = useRoute();
    const getUsersById = async (id) => {
      try {
        const { data } = await api.get(`endereco/${id}`);
        return data;
      } catch (error) {
        throw new Error(error);
      }
    };

    onMounted(async () => {
      if (route.params.id) {
        const response = await getUsersById(route.params.id);
        form.value = response;
        console.log('form', response);
      }
    });

    const $q = useQuasar();
    const router = useRouter();

    // eslint-disable-next-line @typescript-eslint/no-shadow
    const post = async (form) => {
      try {
        const { data } = await api.post('endereco', form.value);
        return data.data;
      } catch (error) {
        throw new Error(error);
      }
    };
    // eslint-disable-next-line @typescript-eslint/no-shadow
    const update = async (form) => {
      try {
        const { data } = await api.put(`endereco/${route.params.id}`, form.value);
        $q.notify({ message: 'Endereço atualizado', icon: 'check', color: 'positive' });
        router.push({ name: 'home' });
        return data;
      } catch (error) {
        throw new Error(error);
      }
    };

    const buscaPorCep = async () => {
      try {
        if (form.value.cep.length === 8) {
          const { data } = await axios.get(`https://viacep.com.br/ws/${form.value.cep}/json/`);
          form.value.rua = data.logradouro;
          form.value.bairro = data.bairro;
        }
      } catch (error) {
        throw new Error(error);
      }
    };

    const handleSubmit = async () => {
      if (route.params.id) {
        try {
          await update(form);
        } catch (error) {
          $q.notify({ message: 'Error: endereco não atualizado', icon: 'times', color: 'negative' });
        }
      } else {
        post(form);
        $q.notify({ message: 'Endereco cadastrado', icon: 'check', color: 'positive' });
        router.push({ name: 'home' });
      }
    };

    return {
      form,
      handleSubmit,
      buscaPorCep,
    };
  },
});
</script>

<style></style>
