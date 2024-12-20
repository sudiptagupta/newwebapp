<template>
  <q-page class="bg-dark row">
    <!-- Left Sidebar - hide on mobile by default -->
    <div class="col-12 col-sm-12 col-md-3 q-pa-md sidebar-container">
      <q-card class="bg-dark-page text-white" bordered>
        <q-card-section>
          <!-- TradingView Widget -->
          <div class="tradingview-widget-container" ref="tradingViewContainer">
            <div class="tradingview-widget-container__widget"></div>
            <div class="tradingview-widget-copyright">
              <a
                href="https://www.tradingview.com/"
                rel="noopener nofollow"
                target="_blank"
              >
                <span class="blue-text">Track all markets on TradingView</span>
              </a>
            </div>
          </div>
        </q-card-section>
      </q-card>
    </div>

    <!-- Main Content - show first on mobile -->
    <div class="col-12 col-sm-12 col-md-9 main-content">
      <!-- Navigation tabs -->
      <div class="q-pa-md">
        <div class="row items-center">
          <q-btn flat round icon="settings" color="grey" class="q-mr-md" />
          <q-tabs
            v-model="tab"
            class="text-grey"
            active-color="white"
            indicator-color="primary"
            align="left"
          >
            <q-tab name="myfeed" label="My Feed" />
            <q-tab name="equity" label="Equity" />
            <q-tab name="crypto" label="Crypto" />
            <q-tab name="forex" label="Forex" />
            <q-tab name="commodities" label="Commodities" />
          </q-tabs>
        </div>
      </div>

      <!-- Feed Items -->
      <div class="q-pa-md">
        <!-- Loading state -->
        <div v-if="loading" class="fullscreen-loader">
          <q-spinner-dots color="primary" size="4em" />
          <div class="text-white q-mt-md">Loading your feed...</div>
        </div>

        <!-- Error state -->
        <div v-else-if="error" class="text-center text-negative">
          {{ error }}
        </div>

        <!-- Feed Items -->
        <template v-else>
          <q-infinite-scroll @load="loadMore" :offset="250">
            <q-card
              v-for="(item, index) in displayedItems"
              :key="index"
              class="q-mb-md bg-dark-page text-white"
              :class="{ 'new-record': item.isNew }"
              bordered
            >
              <q-card-section>
                <div class="row items-center justify-between">
                  <div class="col-auto q-mr-md">
                    <div class="text-caption q-mb-sm">
                      {{ formatDate(item.post_timestamp) }}
                    </div>
                  </div>
                  <div class="col">
                    <div class="row items-center q-mt-sm">
                      <div class="text-h6 col">
                        {{ item.headline }}
                        <q-btn
                          v-if="item.post_url"
                          flat
                          round
                          dense
                          color="primary"
                          icon="launch"
                          type="a"
                          :href="item.post_url"
                          target="_blank"
                          class="col-auto"
                          size="12px"
                        />
                      </div>
                    </div>
                    <div class="text-body2 text-grey">{{ item.summary }}</div>
                    <div class="q-mt-sm">
                      <template v-if="item.tickers">
                        <q-chip
                          v-for="ticker in item.tickers.split(',')"
                          :key="ticker"
                          :label="ticker.trim()"
                          color="amber"
                          text-color="black"
                          class="q-mr-xs"
                          dense
                        />
                      </template>
                    </div>
                  </div>
                  <div class="col-auto">
                    <q-btn flat round color="grey" icon="favorite_border" />
                  </div>
                </div>
              </q-card-section>
            </q-card>

            <template v-slot:loading>
              <div class="row justify-center q-my-md">
                <q-spinner-dots color="primary" size="40px" />
              </div>
            </template>
          </q-infinite-scroll>
        </template>
      </div>
    </div>
  </q-page>
</template>

<script>
import {
  defineComponent,
  ref,
  computed,
  onMounted,
  onUnmounted,
  inject,
  watch,
} from "vue";
import useSupabase from "src/boot/supabase";
import { debounce } from "quasar";

export default defineComponent({
  name: "FeedPage",
  setup() {
    const { supabase } = useSupabase();

    // Inject the search query from MainLayout
    const searchQuery = inject("searchQuery", ref(""));

    // Existing refs
    const tab = ref("myfeed");
    const loading = ref(true);
    const error = ref(null);
    const page = ref(1);
    const itemsPerPage = ref(10);
    const hasMoreItems = ref(true);
    const displayedItems = ref([]);

    // Inject the headlines provided by MainLayout
    const headlines = inject("headlines", ref([]));
    const selectedTicker = inject("selectedTicker", ref(""));

    // Computed property for filtered headlines
    const filteredHeadlines = computed(() => {
      let items = headlines.value;

      // Apply tab filtering except for 'myfeed'
      if (tab.value !== "myfeed") {
        items = items.filter(
          (item) =>
            item.asset_type &&
            item.asset_type.toLowerCase() === tab.value.toLowerCase()
        );
      }

      return items;
    });

    // Watch for changes in filtered headlines
    watch(
      filteredHeadlines,
      (newItems) => {
        page.value = 1;
        displayedItems.value = newItems.slice(0, itemsPerPage.value);
        hasMoreItems.value = newItems.length > itemsPerPage.value;
        loading.value = false;
      },
      { immediate: true }
    );

    // Watch for tab changes
    watch(tab, () => {
      page.value = 1;
      const items = filteredHeadlines.value;
      displayedItems.value = items.slice(0, itemsPerPage.value);
      hasMoreItems.value = items.length > itemsPerPage.value;
    });

    const loadMore = async (index, done) => {
      try {
        const items = filteredHeadlines.value;
        const start = page.value * itemsPerPage.value;
        const end = start + itemsPerPage.value;
        const newItems = items.slice(start, end);

        if (newItems.length > 0) {
          displayedItems.value = [...displayedItems.value, ...newItems];
          page.value++;
          hasMoreItems.value = items.length > end;
        } else {
          hasMoreItems.value = false;
        }
      } finally {
        done();
      }
    };

    // Methods
    const fetchHeadlines = async () => {
      loading.value = true;
      error.value = null;

      try {
        const { data, error: supabaseError } = await supabase
          .from("Headlines")
          .select("*, asset_type")
          .order("post_timestamp", { ascending: false });

        if (supabaseError) throw supabaseError;

        headlines.value = data || [];
        // Reset pagination
        page.value = 1;
        displayedItems.value = [];
        hasMoreItems.value = true;

        // Load initial items
        const initialItems = filteredHeadlines.value.slice(
          0,
          itemsPerPage.value
        );
        displayedItems.value = initialItems;
      } catch (err) {
        error.value = err.message;
        console.error("Error fetching headlines:", err.message);
      } finally {
        loading.value = false;
      }
    };

    // Update the subscribeToHeadlines function
    const subscribeToHeadlines = () => {
      const subscription = supabase
        .from("Headlines")
        .on("*", (payload) => {
          console.log("Real-time update:", payload);

          switch (payload.eventType) {
            case "INSERT":
              // Add isNew flag to the new record
              const newRecord = { ...payload.new, isNew: true };

              // Update both feedItems and displayedItems
              headlines.value = [newRecord, ...headlines.value];

              // Only add to displayedItems if it matches current tab filter
              if (
                tab.value === "myfeed" ||
                (newRecord.asset_type &&
                  newRecord.asset_type.toLowerCase() ===
                    tab.value.toLowerCase())
              ) {
                displayedItems.value = [newRecord, ...displayedItems.value];
              }

              // Remove the isNew flag after 5 seconds
              setTimeout(() => {
                // Update in feedItems
                const feedIndex = headlines.value.findIndex(
                  (item) => item.id === newRecord.id
                );
                if (feedIndex !== -1) {
                  headlines.value[feedIndex] = {
                    ...headlines.value[feedIndex],
                    isNew: false,
                  };
                }

                // Update in displayedItems
                const displayIndex = displayedItems.value.findIndex(
                  (item) => item.id === newRecord.id
                );
                if (displayIndex !== -1) {
                  displayedItems.value[displayIndex] = {
                    ...displayedItems.value[displayIndex],
                    isNew: false,
                  };
                }
              }, 10000);
              break;

            case "DELETE":
              // Remove from both arrays
              headlines.value = headlines.value.filter(
                (item) => item.id !== payload.old.id
              );
              displayedItems.value = displayedItems.value.filter(
                (item) => item.id !== payload.old.id
              );
              break;

            case "UPDATE":
              // Update in both arrays
              const updateFeedIndex = headlines.value.findIndex(
                (item) => item.id === payload.new.id
              );
              if (updateFeedIndex !== -1) {
                headlines.value[updateFeedIndex] = payload.new;
              }

              const updateDisplayIndex = displayedItems.value.findIndex(
                (item) => item.id === payload.new.id
              );
              if (updateDisplayIndex !== -1) {
                displayedItems.value[updateDisplayIndex] = payload.new;
              }
              break;
          }
        })
        .subscribe();

      return subscription;
    };

    // Lifecycle hooks
    onMounted(async () => {
      try {
        await fetchHeadlines();
      } finally {
        loading.value = false;
      }

      const subscription = subscribeToHeadlines();

      // Clean up subscription on unmount
      onUnmounted(() => {
        subscription.unsubscribe();
      });

      const script = document.createElement("script");
      script.src =
        "https://s3.tradingview.com/external-embedding/embed-widget-events.js";
      script.async = true;
      script.innerHTML = JSON.stringify({
        width: "100%",
        height: "100%",
        colorTheme: "dark",
        isTransparent: false,
        locale: "en",
        importanceFilter: "-1,0,1",
        countryFilter:
          "ar,au,br,ca,cn,fr,de,in,id,it,jp,kr,mx,ru,sa,za,tr,gb,us,eu",
      });

      const container = document.querySelector(".tradingview-widget-container");
      container.appendChild(script);
    });

    // Add formatDate function
    const formatDate = (timestamp) => {
      if (!timestamp) return "";
      const date = new Date(timestamp);
      return date.toLocaleString("en-US", {
        month: "short",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
        hour12: true,
      });
    };

    // Add getSentimentColor function
    const getSentimentColor = (sentiment) => {
      const colors = {
        Positive: "positive",
        Negative: "negative",
        Neutral: "grey",
      };
      return colors[sentiment] || "grey";
    };

    return {
      // Existing returns
      tab,
      loading,
      error,
      displayedItems,
      loadMore,
      formatDate,
      selectedTicker,

      // New returns
      page,
      itemsPerPage,
      hasMoreItems,
      filteredHeadlines,
    };
  },
});
</script>

<style lang="scss">
.bg-dark-page {
  background: #1e1e1e;
}

.q-card {
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  transition: border-color 0.5s ease-out;

  &.new-record {
    border: 2px solid $primary;
    animation: highlightFade 10s forwards;
  }
}

.q-tabs {
  background: transparent;
}

/* Responsive sidebar styles */
@media (min-width: 1024px) {
  .col-md-3 {
    border-right: 1px solid rgba(255, 255, 255, 0.1);
  }
}

.q-list {
  .q-item {
    color: #fff;
    border-radius: 8px;

    &:hover {
      background: rgba(255, 255, 255, 0.1);
    }
  }
}

/* Make tabs scrollable on small screens */

.q-card {
  .q-card__section {
    padding: 16px;
  }

  .text-caption {
    font-size: 0.85rem;
  }

  .q-chip {
    height: 24px;
    font-size: 0.85rem;
  }

  .text-h6 {
    margin: 0;
    line-height: 1.5;
  }
}

.blue-text {
  color: #2962ff;
}

.tradingview-widget-container {
  height: 85vh !important;
  width: 100%;
}

.col-12.col-sm-4.col-md-3.q-pa-md {
  height: 100vh;
}

@media (max-width: 599px) {
  .tradingview-widget-container {
    height: 400px;
  }
}

.fullscreen-loader {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.7);
  z-index: 2000;
}

@keyframes highlightFade {
  0% {
    border-color: $primary;
  }
  100% {
    border-color: rgba(255, 255, 255, 0.1);
  }
}

/* Add these new styles */
@media (max-width: 1023px) {
  .q-page {
    display: flex;
    flex-direction: column;
  }

  .sidebar-container {
    order: 2; /* Push sidebar to bottom on mobile */
  }

  .main-content {
    order: 1; /* Show main content first on mobile */
  }

  .tradingview-widget-container {
    height: 400px !important; /* Reduce height on mobile */
  }
}
</style>
