// src/app/app.component.ts
import { Component } from '@angular/core';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterModule],
  template: `
    <div class="container mt-4">
      <h1 class="text-center mb-4">KYC Management System</h1>
      <nav class="mb-4">
        <a routerLink="/register" class="btn btn-outline-primary me-2">Register</a>
        <a routerLink="/submit-kyc" class="btn btn-outline-primary me-2">Submit KYC</a>
        <a routerLink="/kyc-status" class="btn btn-outline-primary me-2">KYC Status</a>
        <a routerLink="/admin-dashboard" class="btn btn-outline-danger">Admin</a>
      </nav>
      <router-outlet></router-outlet>
    </div>
  `
})
export class AppComponent {}
