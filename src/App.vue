<script lang="ts">
import { computed, defineComponent, ref, reactive, watch } from "@vue/composition-api";
import { useQuery, useQueryProvider, useQueries } from "vue-query";
import { VueQueryDevTools } from "vue-query/devtools";

interface Todo {
  userId: number;
  id: number;
  title: string;
  completed: boolean;
}

const todoFetcher = (id) => async (id): Promise<Todo[]> =>
  await fetch("https://jsonplaceholder.cypress.io/todos/id").then((response) =>
    response.json()
  );

function useMultipleTodos() {
  const ids = ref([]);
  const queries = computed(() => ids.value.map((id) => ({
    queryKey: ['post', reactive({ id })],
    queryFn: todoFetcher(id),
  })));
  const queryResults = useQueries(queries.value);
  return reactive({
    queryResults,
    ids,
    queries,
    fetch: (_ids) => { ids.value = _ids; }
  });
}

export default defineComponent({
  name: "App",
  components: { VueQueryDevTools },
  setup() {
    useQueryProvider();
    return {
      multipleTodos: useMultipleTodos(),
    };
  },
  methods: {
    onFetchMultipleTodos() {
      this.multipleTodos.fetch([1,2,3]);
    }
  }
});
</script>

<template>
  <div>
    <button @click="onFetchMultipleTodos">Fetch multiple todos</button>
    <div>{{ multipleTodos.ids }}</div>
    <div>{{ multipleTodos.queries }}</div>
    <VueQueryDevTools :initialIsOpen="true" />
  </div>
</template>
