<template>
  <v-app>
    <v-app-bar app color="gray" flat>
      <v-tabs color="grey darken-1" v-if="user && user.isAdmin">
        <v-tab v-for="[link, href] in links" :key="link" :to="href">
          {{ link }}
        </v-tab>
      </v-tabs>

      <!-- <v-btn elevation="2" button  :color="'transparent'" size="32"> test</v-btn> -->
      <v-spacer/>
      <h3>{{ user ? user.email : "Гість" }}</h3>
      <v-btn icon x-large @click="profileClick">
        <v-avatar size="32">
          <v-icon dark>
            {{
              user && user.isAdmin && "mdi-account-cowboy-hat" || "mdi-account-circle"
            }}
          </v-icon>
        </v-avatar>
      </v-btn>
    </v-app-bar>

    <v-main class="grey lighten-3">
      <v-container>
        <router-view />
      </v-container>
    </v-main>

    <v-dialog v-model="logoutDialog" max-width="290">
      <v-card>
        <v-card-title class="text-h5"> Підтвердження </v-card-title>
        <v-card-text>Ви правда хочете вийти з акаунта?</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" text @click="logoutDialog = false">
            Ні
          </v-btn>
          <v-btn
            color="green darken-1"
            text
            @click="
              logoutDialog = false;
              logout();
            "
          >
            Так
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="loginDialog" max-width="290">
      <v-card class="elevation-12">
        <v-toolbar dark color="primary">
          <v-toolbar-title>
            {{ registerView ? "Реєстрація" : "Вхід" }}
          </v-toolbar-title>
        </v-toolbar>
        <v-card-text>
          <v-form v-if="!registerView">
            <v-text-field
              prepend-icon="mdi-account"
              v-model="email"
              :rules="emailRule"
              label="Пошта"
              type="email"
            ></v-text-field>
            <v-text-field
              prepend-icon="mdi-lock"
              v-model="password"
              label="Пароль"
              type="password"
            ></v-text-field>
          </v-form>

          <v-form v-else>
            <v-text-field
              prepend-icon="mdi-account"
              v-model="email"
              :rules="emailRule"
              label="Пошта"
              type="email"
            ></v-text-field>
            <v-text-field
              prepend-icon="mdi-lock"
              v-model="password"
              label="Пароль"
              type="password"
            ></v-text-field>
            <v-text-field
              prepend-icon="mdi-lock"
              v-model="password_confirm"
              :rules="confirmPasswordRule"
              label="Підтвердження пароля"
              type="password"
            ></v-text-field>
          </v-form>
        </v-card-text>
        <h3 class="red--text text--lighten-1" v-if="loginError">
          {{ loginError }}
        </h3>
        <v-card-actions>
          <v-btn @click="registerView = !registerView; loginError=''">{{
            registerView ? "Вже є" : "Створити"
          }}</v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="loginClick">{{
            registerView ? "Створити" : "Зайти"
          }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
export default {
  name: "Home",
  async created() {
    await this.$store.dispatch("getUser");
  },
  computed: {
    user() {
      return this.$store.state.user;
    },
    emailRule( ) {
      return [
        text => {
          return (/\S+@\S+\.\S+/).test( text ) || "Неправильный формат почти"
        }
      ]
    },
    confirmPasswordRule( ) {
      return [
        text => {
          return text == this.password || "Пароли не співпадають"
        }
      ]
    }
  },
  data() {
    return {
      email: "",
      password: "",
      password_confirm: "",
      loginError: false,
      loginDialog: false,
      registerView: false,
      logoutDialog: false,
      links: [
        ["Головна сторінка", "/"],
        ["Створити пост", "/newPost"],
      ],
    };
  },
  methods: {
    async logout() {
      await this.$store.dispatch("logout");
      this.$router.go("");
    },
    profileClick() {
      if (this.user) {
        this.logoutDialog = true;
      } else {
        this.loginDialog = true;
      }
    },
    async loginClick() {
      if( typeof this.emailRule[0]( this.email ) == "string" ){
        return;
      }
      if (this.registerView) {
        if (this.password != this.password_confirm) {
          return;
        }
        const reigsterResult = await this.$store.dispatch("register", {
          email: this.email,
          password: this.password,
        });
        if (!reigsterResult) {
          this.loginError = "Неправильний логін або пароль";
        } else {
          this.$router.go("/");
        }
      } else {
        const loginResult = await this.$store.dispatch("login", {
          email: this.email,
          password: this.password,
        });
        console.log( loginResult );
        if (!loginResult) {
          this.loginError = "Неправильний логін або пароль";
        }else {
          this.$router.go("")
        }
      }
    },
  },
};
</script>



<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
