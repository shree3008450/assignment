// src/app/app.routes.ts
import { Routes } from '@angular/router';
import { CustomerRegisterComponent } from './customer-register/customer-register.component';
import { KycFormComponent } from './kyc-form/kyc-form.component';
import { KycStatusComponent } from './kyc-status/kyc-status.component';
import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';

export const routes: Routes = [
  { path: '', redirectTo: 'register', pathMatch: 'full' },
  { path: 'register', component: CustomerRegisterComponent },
  { path: 'submit-kyc', component: KycFormComponent },
  { path: 'kyc-status', component: KycStatusComponent },
  { path: 'admin-dashboard', component: AdminDashboardComponent },
];
