﻿<template>
  <section class="section section-login">
    <div class="container has-text-centered">
      <div class="column is-4 is-offset-4">
        <h3 class="title">Login</h3>
        <p class="subtitle">Please login to proceed</p>
        <div class="box login-box">
          <img
            width="175"
            id="login-img"
            alt="based-ghost-logo"
            src="@/assets/img/based-ghost-main.png"
          />
          <form @submit.prevent="handleLogin">
            <user-name-input :isInputInvalid="invalidInputs" />
            <password-input :isInputInvalid="invalidInputs" />
            <remember-me-input />
            <button type="submit" class="button is-info is-medium is-fullwidth">
              <span>Login</span>
              <span class="icon">
                <font-awesome-icon icon="sign-in-alt" />
              </span>
            </button>
            <authenticator
              :authStatus="authStatus"
              @success="onAuthSuccess"
              @fail="onAuthFailure"
            />
          </form>
        </div>
      </div>
    </div>
  </section>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { SignalRApi } from "@/api";
import { Authenticator } from "@/components";
import { AuthModule, AuthStatusEnum } from "@/store/modules/auth";
import { UserNameInput, PasswordInput, RememberMeInput } from "./child-components";

@Component({
  components: {
    Authenticator,
    UserNameInput,
    PasswordInput,
    RememberMeInput
  }
})
export default class Login extends Vue {
  public invalidInputs: boolean = false;
  public authStatus: AuthStatusEnum = AuthStatusEnum.NONE;

  get isUserNameOrPasswordEmpty(): boolean {
    return AuthModule.isUserNameOrPasswordEmpty;
  }

  public mounted(): void {
    SignalRApi.startConnection();
  }

  public handleLogin(): void {
    if (this.authStatus === AuthStatusEnum.PROCESS) {
      return;
    }

    if (this.isUserNameOrPasswordEmpty) {
      this.invalidInputs = true;
      this.$snotify.error('Enter user name/password', 'Login Error');
      return;
    }

    this.$snotify.clear();
    this.invalidInputs = false;
    this.authStatus = AuthStatusEnum.PROCESS;

    setTimeout(() => {
      AuthModule.LoginUser()
        .then(() => {
          this.authStatus = AuthStatusEnum.SUCCESS;
        })
        .catch((e) => {
          this.authStatus = AuthStatusEnum.FAIL;
        });
    }, 2250);
  }

  public onAuthSuccess(): void {
    this.$router.push('/dashboard');
  }

  public onAuthFailure(): void {
    this.authStatus = AuthStatusEnum.NONE;
    this.$snotify.error('Could not authenticate user', 'Login Error');
  }
}
</script>