<template>
  <app-loader v-if="loading"></app-loader>
  <app-page title="Заявка" back v-else-if="request">
    <p><strong>Имя владельца</strong>: {{request.fio}}</p>
    <p><strong>Телефон</strong>: {{request.phone}}</p>
    <p><strong>Сумма</strong>: {{currency(request.amount)}}</p>
    <p><strong>Статус</strong>: <app-status :type="request.status" /></p>

    <div class="form-control">
      <label for="status">Статус</label>
      <select id="status" v-model="status">
        <option value="done">Завершен</option>
        <option value="cancelled">Отменен</option>
        <option value="active">Активен</option>
        <option value="pending">Выполняется</option>
      </select>
    </div>

    <button class="btn danger" @click="remove">Удалить</button>
    <button class="btn" @click="update" v-if="hasChanges">Обновить</button>
  </app-page>
  <h3 class="text-center text-white" v-else>
    Заявки с id {{$route.params.id}} нет!
  </h3>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useStore } from 'vuex';
import AppPage from '../components/ui/AppPage.vue';
import AppLoader from '../components/ui/AppLoader.vue';
import AppStatus from '../components/ui/AppStatus.vue';
import { currency } from '../utils/currency';
export default {
  components: {
    AppPage,
    AppLoader,
    AppStatus
  },
  setup() {
    const route = useRoute();
    const router = useRouter();
    const store = useStore();
    const loading = ref(true);
    const request = ref({});
    const status = ref();

    onMounted(async () => {
      loading.value = true;
      request.value = await store.dispatch('request/loadOne', route.params.id);
      status.value = request.value?.status;
      loading.value = false;
    });

    const hasChanges = computed(() => request.value.status !== status.value);

    const remove = async () => {
      await store.dispatch('request/remove', route.params.id);
      router.push('/');
    };

    const update = async () => {
      const data = {...request.value, status: status.value, id: route.params.id};
      await store.dispatch('request/update', data);
      request.value.status = status.value;
    };

    return {
      loading,
      request,
      currency,
      remove,
      update,
      status,
      hasChanges
    }
  }
};
</script>
