<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="bg-dark text-white">
      <q-toolbar>
        <q-toolbar-title class="cursor-pointer" @click="goHome">
          Traddio
        </q-toolbar-title>

        <div class="row justify-center col-grow">
          <div class="col-12 col-sm-8 col-md-6 search-container">
            <div v-if="selectedTicker" class="selected-company">
              <q-chip
                removable
                @remove="clearSelection"
                color="primary"
                text-color="white"
              >
                {{ selectedTicker }}
              </q-chip>
            </div>

            <q-input
              v-model="searchText"
              dense
              outlined
              dark
              :disable="!!selectedTicker"
              placeholder="Search anything i.e APPLE or BTC"
              class="search-input"
              @update:model-value="filterCompanies"
              @input="handleSearchClear"
              @keydown.down.prevent="navigateResults('down')"
              @keydown.up.prevent="navigateResults('up')"
              @keydown.enter.prevent="selectHighlighted"
            >
              <template v-slot:append>
                <q-icon name="search" />
              </template>
            </q-input>

            <q-list
              v-if="showResults && companyOptions.length > 0"
              class="search-results bg-dark"
              bordered
              ref="searchResults"
            >
              <q-item
                v-for="(company, index) in companyOptions"
                :key="company.id"
                clickable
                v-ripple
                @click="handleCompanySelect(company)"
                :class="{ highlighted: index === highlightedIndex }"
              >
                <q-item-section>
                  <q-item-label>{{ company.company_name }}</q-item-label>
                  <q-item-label class="text-body2 text-grey" caption>{{
                    company.company_ticker
                  }}</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
          </div>
        </div>

        <q-btn-dropdown flat color="white" icon="person">
          <q-list>
            <q-item
              clickable
              v-close-popup
              @click="handlerLogout"
              class="text-black"
            >
              <q-item-section>
                <q-item-label>Logout</q-item-label>
              </q-item-section>
            </q-item>
          </q-list>
        </q-btn-dropdown>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import {
  defineComponent,
  ref,
  provide,
  onMounted,
  onBeforeUnmount,
  watch,
  nextTick,
} from "vue";
import { useRouter } from "vue-router";
import useSupabase from "src/boot/supabase";
import { debounce } from "quasar";

import userAuthUser from "../composables/UserAuthUser";
import useNotify from "src/composables/UseNotify";
import useDialog from "src/composables/UseDialog";

export default defineComponent({
  name: "MainLayout",

  setup() {
    const router = useRouter();
    const { logout } = userAuthUser();
    const { notifyError, notifySuccess } = useNotify();
    const { dialogShow } = useDialog();
    const { supabase } = useSupabase();
    const searchText = ref("");
    const companyOptions = ref([]);
    const showResults = ref(false);
    const selectedTicker = ref("");
    const headlines = ref([]);
    const highlightedIndex = ref(-1);
    const searchResults = ref(null);

    // Add function to fetch all headlines
    const fetchAllHeadlines = async () => {
      try {
        const { data, error } = await supabase
          .from("Headlines")
          .select("*")
          .order("post_timestamp", { ascending: false });

        if (error) throw error;

        headlines.value = data || [];
      } catch (error) {
        console.error("Error fetching initial headlines:", error);
        headlines.value = [];
      }
    };

    // Load headlines when component mounts
    onMounted(async () => {
      await fetchAllHeadlines();
      document.addEventListener("click", handleClickOutside);
    });

    // Create debounced search function
    const debouncedSearch = debounce(async (searchValue) => {
      console.log("Searching for:", searchValue); // Debug log

      if (!searchValue || searchValue.trim() === "") {
        companyOptions.value = [];
        showResults.value = false;
        return;
      }

      try {
        // Modified query to use ilike with proper syntax
        const { data, error } = await supabase
          .from("Companies")
          .select("id, company_name, company_ticker")
          .or(
            `company_name.ilike.%${searchValue}%,company_ticker.ilike.%${searchValue}%`
          )
          .limit(10);

        console.log("Query result:", { data, error }); // Debug log

        if (error) {
          console.error("Supabase error:", error); // Debug log
          throw error;
        }

        if (data) {
          companyOptions.value = data;
          showResults.value = true;
          console.log("Found companies:", data); // Debug log
        }
      } catch (error) {
        console.error("Search error:", error);
        companyOptions.value = [];
        showResults.value = false;
      }
    }, 300);

    // Wrapper function for the debounced search
    const filterCompanies = (value) => {
      debouncedSearch(value);
    };

    const handleCompanySelect = async (company) => {
      try {
        // Update to show ticker instead of company name
        searchText.value = company.company_ticker;
        selectedTicker.value = company.company_ticker;
        showResults.value = false;

        const { data, error } = await supabase
          .from("Headlines")
          .select("*")
          .or(`tickers.ilike.%${company.company_ticker}%`)
          .order("post_timestamp", { ascending: false });

        if (error) throw error;
        headlines.value = data || [];
        console.log(
          "Found headlines for ticker:",
          company.company_ticker,
          data?.length
        );
      } catch (error) {
        console.error("Error fetching headlines:", error);
        headlines.value = [];
      }
    };

    // Click outside to close results
    const handleClickOutside = (event) => {
      const searchContainer = document.querySelector(".search-container");
      if (searchContainer && !searchContainer.contains(event.target)) {
        showResults.value = false;
      }
    };

    onBeforeUnmount(() => {
      document.removeEventListener("click", handleClickOutside);
    });

    // Provide the selected ticker and headlines to child components
    provide("selectedTicker", selectedTicker);
    provide("headlines", headlines);

    const handlerLogout = async () => {
      dialogShow({
        tittle: "To go out",
        message: "Are you sure you want to exit the application?",
        class: "text-white",
        dark: true,
        persistent: true,
        ok: {
          push: true,
          color: "primary",
          label: "Yes",
        },
        cancel: {
          push: true,
          color: "negative",
          label: "No",
        },
      })
        .onOk(async () => {
          try {
            await logout();
            notifySuccess("Bye bye!");
            router.replace({
              name: "login",
            });
          } catch (error) {
            notifyError(error.message);
          }
        })
        .onCancel(async () => {
          notifySuccess("Oops, I was going to but I came back!");
        });
    };

    // Update handleSearchClear function
    const handleSearchClear = async () => {
      console.log("handleSearchClear called");
      if (!searchText.value || !searchText.value.trim()) {
        selectedTicker.value = "";
        await fetchAllHeadlines();
      }
    };

    // Add clearSelection function
    const clearSelection = async () => {
      selectedTicker.value = "";
      searchText.value = "";
      await fetchAllHeadlines();
    };

    const scrollToHighlighted = () => {
      if (!searchResults.value) return;

      // Fix: Access the DOM element using $el
      const highlightedElement =
        searchResults.value.$el.querySelector(".highlighted");
      if (highlightedElement) {
        highlightedElement.scrollIntoView({
          block: "nearest",
          behavior: "smooth",
        });
      }
    };

    const navigateResults = (direction) => {
      if (!showResults.value || companyOptions.value.length === 0) return;

      if (direction === "down") {
        highlightedIndex.value =
          highlightedIndex.value < companyOptions.value.length - 1
            ? highlightedIndex.value + 1
            : 0;
      } else {
        highlightedIndex.value =
          highlightedIndex.value > 0
            ? highlightedIndex.value - 1
            : companyOptions.value.length - 1;
      }

      // Add scroll after updating the highlight
      nextTick(() => scrollToHighlighted());
    };

    const selectHighlighted = () => {
      if (
        highlightedIndex.value >= 0 &&
        companyOptions.value[highlightedIndex.value]
      ) {
        handleCompanySelect(companyOptions.value[highlightedIndex.value]);
      }
    };

    // Reset highlighted index when search results change
    watch(companyOptions, () => {
      highlightedIndex.value = -1;
    });

    // Reset highlighted index when closing results
    watch(showResults, (newValue) => {
      if (!newValue) highlightedIndex.value = -1;
    });

    return {
      handlerLogout,
      searchText,
      companyOptions,
      showResults,
      filterCompanies,
      handleCompanySelect,
      handleSearchClear,
      selectedTicker,
      clearSelection,
      highlightedIndex,
      navigateResults,
      selectHighlighted,
      searchResults,
    };
  },
});
</script>

<style lang="scss">
.col-grow {
  flex-grow: 1;
  padding: 0 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.search-container {
  position: relative;
}

.search-input {
  width: 100%;
  max-width: 600px;

  .q-field__control {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 20px;

    &:hover {
      background: rgba(255, 255, 255, 0.08);
    }
  }

  .q-field__marginal {
    color: rgba(255, 255, 255, 0.7);
  }

  input {
    color: white !important;
    &::placeholder {
      color: rgba(255, 255, 255, 0.5);
    }
  }
}

.search-results {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  z-index: 1000;
  margin-top: 4px;
  border-radius: 4px;
  max-height: 300px;
  overflow-y: auto;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);

  .q-item {
    padding: 8px 16px;

    &:hover {
      background: rgba(255, 255, 255, 0.1);
    }

    &.highlighted {
      background: rgba(255, 255, 255, 0.2);
    }
  }

  .q-item-label {
    color: white;

    &.caption {
      color: rgba(255, 255, 255, 0.7);
    }
  }
}

.q-toolbar {
  padding: 0 20px;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.q-toolbar__title {
  flex: 0 0 auto;
}

@media (max-width: 599px) {
  .search-input {
    width: 100%;
  }

  .q-toolbar {
    padding: 0 10px;
  }

  .col-grow {
    padding: 0 10px;
  }
}

// Add styles for dialog
:deep(.q-dialog__title),
:deep(.q-dialog__message) {
  color: white !important;
}

.q-dialog {
  .q-card {
    background: #1e1e1e;
  }
}

// Add styles for the selected company chip
.selected-company {
  position: absolute;
  top: 50%;
  left: 12px;
  transform: translateY(-50%);
  z-index: 2;
  pointer-events: auto;

  .q-chip {
    font-size: 14px;
    margin: 0;
  }
}

// Adjust input padding when chip is present
.q-field--dense.q-field--outlined .q-field__control:has(+ .selected-company) {
  padding-left: 120px; // Adjust this value based on your chip width
}
</style>
