<template>
  <div class="container">
    <div class="panel">
      <h3>Available recipients</h3>
      <a-input v-model:value="searchQuery" placeholder="Search recipients" @input="filterSuggestions">
        <template #prefix>
          <SearchOutlined />
        </template>
      </a-input>

      <a-list v-if="suggestions.length && searchQuery" bordered class="autocomplete">
        <a-list-item class="cursor-pointer" v-for="suggestion in suggestions" :key="suggestion" @click="(e)=>selectRecipient(e,suggestion)">
          {{ suggestion }}
        </a-list-item>
      </a-list>

      <a-list class="recipients-list">
        <template v-for="(emails, company) in filteredRecipients" :key="company">
          <a-list-item 
          >
            <div class="company" @click="toggleCompany(company)">
              <span>{{ isOpen(company) ? '▼' : '▶' }}</span>
              <span class="company-name mrgin-left" @click.stop="selectCompany(company)">{{ company.split('.')[0].toUpperCase() }}</span>
            </div>
            <div v-if="isOpen(company)" class="emails">
              <a-list-item class="email" v-for="email in emails" :key="email" @click="(e) => selectRecipient(e,email)">
                {{ email }}
              </a-list-item>
            </div>
          </a-list-item>
        </template>
      </a-list>
    </div>

    <div class="panel">
      <h3>Selected recipients</h3>
      <a-list class="recipients-list">
        <template v-for="(emails, company) in selectedRecipients" :key="company">
          <a-list-item>
            <div class="group-header" @click="toggleSelectedGroup(company)">
              <span>{{ isOpenSelected(company) ? '▼' : '▶' }}</span>
              <span class="company-name mrgin-left">{{ company.split('.')[0].toUpperCase() }}</span>
              <a-button class="margin-left" type="dashed" danger @click.stop="removeCompany(company,emails)">
                <DeleteOutlined />
                Remove All
              </a-button>
            </div>
            <div v-if="isOpenSelected(company)" class="emails">
              <a-list-item v-for="email in emails" :key="email">
                {{ email }}
                <a-button class="margin-left" type="dashed" danger @click.stop="removeRecipient(email)">
                  <DeleteOutlined />
                  Remove
                </a-button>
              </a-list-item>
            </div>
          </a-list-item>
        </template>
      </a-list>
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";
import { Input, List, Button } from "ant-design-vue";
import { SearchOutlined,DeleteOutlined } from "@ant-design/icons-vue";

export default defineComponent({
  components: {
    AInput: Input,
    AList: List,
    AListItem: List.Item,
    AButton: Button,
    SearchOutlined,DeleteOutlined,
  },
  data() {
    return {
      searchQuery: "",
      openCompanies: new Set(),
      openSelectedGroups: new Set(),
      availableRecipients: {},
      selectedRecipients: {},
      suggestions: [],
    };
  },
  created() {
    const recipientData = [
      { email: "ann@timescale.com", isSelected: false },
      { email: "bob@timescale.com", isSelected: false },
      { email: "brian@qwerty.com", isSelected: false },
      { email: "james@qwerty.com", isSelected: false },
      { email: "jane@awesome.com", isSelected: false },
      { email: "kate@qwerty.com", isSelected: false },
      { email: "mike@hello.com", isSelected: false },
    ];

    recipientData.forEach(({ email, isSelected }) => {
      let domain = email.split("@")[1];
      if (!this.availableRecipients[domain]) {
        this.availableRecipients[domain] = [];
      }
      this.availableRecipients[domain].push(email);
      
      if (isSelected) {
        if (!this.selectedRecipients[domain]) {
          this.selectedRecipients[domain] = [];
        }
        this.selectedRecipients[domain].push(email);
      }
    });
  },
  computed: {
    filteredRecipients() {
      if (!this.searchQuery) return this.availableRecipients;
      let query = this.searchQuery.toLowerCase();
      let filtered = {};
      for (let company in this.availableRecipients) {
        let emails = this.availableRecipients[company].filter((email) =>
          email.includes(query)
        );
        if (emails.length){ 
        filtered[company] = emails;
        }
      }
      return filtered;
    },
  },
  methods: {
    toggleCompany(company) {
      this.openCompanies.has(company)
        ? this.openCompanies.delete(company)
        : this.openCompanies.add(company);
    },
    isOpen(company) {
      return this.openCompanies.has(company);
    },
    selectRecipient(e,email) {
      e.stopPropagation();
      let domain = email.split("@")[1];
      if (!this.selectedRecipients[domain]) {
        this.selectedRecipients[domain] = [];
      }
       this.filteredRecipients[domain] = this.availableRecipients[domain].filter(
        (e) => e !== email
      );
      if (this.filteredRecipients[domain].length === 0) {
        delete this.filteredRecipients[domain];
      }
      if (!this.selectedRecipients[domain].includes(email)) {
        this.selectedRecipients[domain].push(email);
      }
      const index = this.suggestions.indexOf(email);
      if (index > -1) {
        this.suggestions.splice(index, 1);
      }
    },
    selectCompany(company) {
      if (!this.selectedRecipients[company]) {
        this.selectedRecipients[company] = [...this.availableRecipients[company]];
      }
      delete this.availableRecipients[company];
    },
    toggleSelectedGroup(company) {
      this.openSelectedGroups.has(company)
        ? this.openSelectedGroups.delete(company)
        : this.openSelectedGroups.add(company);
    },
    isOpenSelected(company) {
      return this.openSelectedGroups.has(company);
    },
    removeRecipient(email) {
      let domain = email.split("@")[1];
      this.selectedRecipients[domain] = this.selectedRecipients[domain].filter(
        (e) => e !== email
      );
      if (!this.availableRecipients[domain]) {
        this.availableRecipients[domain] = [];
        this.availableRecipients[domain].push(email);
      }else{
        const index = this.availableRecipients[domain].indexOf(email);
        if (index < 0) {
          this.availableRecipients[domain].push(email);
        }
      }
      if (this.selectedRecipients[domain].length === 0) {
        delete this.selectedRecipients[domain];
      }
       if (!this.filteredRecipients[domain].includes(email)) {
        this.filteredRecipients[domain].push(email);
      }
    if(this.searchQuery) this.filterSuggestions(this.searchQuery);
    },
    removeCompany(company,emails) {
      if (!this.availableRecipients[company]) {
        this.availableRecipients[company] = [];
        this.availableRecipients[company].push(...emails);
      }
      else{
        const filtered = emails.filter((email) => !this.availableRecipients[company].includes(email));
        this.availableRecipients[company].push(...filtered);
      }

      if(this.searchQuery) this.filterSuggestions(this.searchQuery);
      delete this.selectedRecipients[company];
    },
    filterSuggestions() {
      let query = this.searchQuery.toLowerCase();
      let allEmails = Object.values(this.availableRecipients).flat();
      
      this.suggestions = allEmails.filter((email) =>
        email.includes(query || query.split("@")[1])
      );
      if (/^[\w.-]+@[\w.-]+\.[a-zA-Z]{2,}$/.test(this.searchQuery)) {
        if (!this.suggestions.includes(this.searchQuery)){
          this.suggestions.push(this.searchQuery);
        }
      }
    },
    addRecipient(email) {
      let domain = email.split("@")[1];
      if (!this.availableRecipients[domain]) {
        this.availableRecipients[domain] = [];
      }
      if (!this.availableRecipients[domain].includes(email)) {
        this.availableRecipients[domain].push(email);
      }
      this.searchQuery = "";
      this.suggestions = [];
    },
  },
});
</script>

<style scoped>
.container {
  display: flex;
  gap: 20px;
  width: 100%;
}
.panel {
  border: 1px solid #ccc;
  padding: 10px;
  width: 50%;
  border-radius: 12px;
  background-color: #cccccc2e;
}
.autocomplete {
  background: white;
  max-height: 150px;
  overflow-y: auto;
}
.company, .group-header {
  cursor: pointer;
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: flex-start;
}
.emails {
  margin-left: 15px;
}
.email {
  cursor: pointer;
}
.ant-list .ant-list-item{
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-start;
}
.margin-left{
  margin-left: 10px;
}
.cursor-pointer{
  cursor: pointer;
}
@media (max-width: 600px) {
  .container {
    flex-direction: column;
    gap: 10px;
  }
  .panel {
    width: 100%;
  }
}
</style>
