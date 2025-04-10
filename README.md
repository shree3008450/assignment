<div class="container mt-4">
  <h2>Register Customer</h2>
  <form (ngSubmit)="register()" [formGroup]="customerForm">
    <div class="mb-3">
      <label>Name</label>
      <input type="text" class="form-control" formControlName="name" required>
    </div>
    <div class="mb-3">
      <label>Email</label>
      <input type="email" class="form-control" formControlName="email" required>
    </div>
    <div class="mb-3">
      <label>Address</label>
      <input type="text" class="form-control" formControlName="address" required>
    </div>
    <div class="mb-3">
      <label>Phone Number</label>
      <input type="text" class="form-control" formControlName="phoneNumber" required>
    </div>
    <div class="mb-3">
      <label>Date of Birth</label>
      <input type="date" class="form-control" formControlName="dateOfBirth" required>
    </div>
    <button type="submit" class="btn btn-primary">Register</button>
  </form>
</div>
import { Component } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { KycService } from '../services/kyc.service';

@Component({
  selector: 'app-customer-register',
  standalone: true,
  templateUrl: './customer-register.component.html',
  styleUrls: ['./customer-register.component.css'],
})
export class CustomerRegisterComponent {
  customerForm: FormGroup;

  constructor(private fb: FormBuilder, private kycService: KycService) {
    this.customerForm = this.fb.group({
      name: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]],
      address: ['', Validators.required],
      phoneNumber: ['', Validators.required],
      dateOfBirth: ['', Validators.required],
    });
  }

  register(): void {
    if (this.customerForm.valid) {
      this.kycService.registerCustomer(this.customerForm.value).subscribe({
        next: (res) => {
          alert('Customer registered successfully!');
          localStorage.setItem('customerId', res.id);
        },
        error: (err) => console.error('Error registering:', err),
      });
    }
  }
}
<div class="container mt-4">
  <h2>Submit KYC</h2>
  <input type="file" (change)="onFileSelected($event)" class="form-control mb-3" />
  <button class="btn btn-success" (click)="submitKyc()">Submit KYC</button>
</div>
import { Component } from '@angular/core';
import { KycService } from '../services/kyc.service';

@Component({
  selector: 'app-kyc-form',
  standalone: true,
  templateUrl: './kyc-form.component.html',
  styleUrls: ['./kyc-form.component.css'],
})
export class KycFormComponent {
  selectedFile: File | null = null;

  constructor(private kycService: KycService) {}

  onFileSelected(event: any): void {
    this.selectedFile = event.target.files[0];
  }

  submitKyc(): void {
    const customerId = localStorage.getItem('customerId');
    if (!this.selectedFile || !customerId) {
      alert('Please select a file and ensure customer is registered.');
      return;
    }

    const formData = new FormData();
    formData.append('file', this.selectedFile);
    formData.append('customerId', customerId);

    this.kycService.submitKyc(formData).subscribe({
      next: () => alert('KYC submitted!'),
      error: (err) => console.error('Error submitting KYC:', err),
    });
  }
}
<div class="container mt-4">
  <h2>Your KYC Status</h2>
  <div *ngIf="kycStatus">
    <p><strong>Status:</strong> {{ kycStatus.status }}</p>
    <p><strong>Submitted At:</strong> {{ kycStatus.submittedAt }}</p>
    <p><strong>Document:</strong> <a [href]="kycStatus.documentPath" target="_blank">View Document</a></p>
  </div>
  <div *ngIf="!kycStatus">
    <p>No KYC found.</p>
  </div>
</div>
import { Component, OnInit } from '@angular/core';
import { KycService } from '../services/kyc.service';

@Component({
  selector: 'app-kyc-status',
  standalone: true,
  templateUrl: './kyc-status.component.html',
  styleUrls: ['./kyc-status.component.css'],
})
export class KycStatusComponent implements OnInit {
  kycStatus: any;

  constructor(private kycService: KycService) {}

  ngOnInit(): void {
    const customerId = localStorage.getItem('customerId');
    if (customerId) {
      this.kycService.getKycStatus(+customerId).subscribe({
        next: (data) => (this.kycStatus = data),
        error: (err) => console.error('Error fetching KYC status:', err),
      });
    }
  }
}
<div class="container mt-4">
  <h2>Admin Dashboard</h2>
  <div *ngIf="applications.length === 0">
    <p>No pending KYC applications.</p>
  </div>
  <div *ngFor="let app of applications" class="card p-3 mb-3">
    <p><strong>ID:</strong> {{ app.id }}</p>
    <p><strong>Status:</strong> {{ app.status }}</p>
    <p><strong>Submitted:</strong> {{ app.submittedAt }}</p>
    <p><strong>Document:</strong> <a [href]="app.documentPath" target="_blank">View</a></p>
    <button class="btn btn-success me-2" (click)="updateStatus(app.id, 'ACCEPTED')">Accept</button>
    <button class="btn btn-danger" (click)="updateStatus(app.id, 'REJECTED')">Reject</button>
  </div>
</div>
import { Component, OnInit } from '@angular/core';
import { KycService } from '../services/kyc.service';

@Component({
  selector: 'app-admin-dashboard',
  standalone: true,
  templateUrl: './admin-dashboard.component.html',
  styleUrls: ['./admin-dashboard.component.css'],
})
export class AdminDashboardComponent implements OnInit {
  applications: any[] = [];

  constructor(private kycService: KycService) {}

  ngOnInit(): void {
    this.fetchApplications();
  }

  fetchApplications(): void {
    this.kycService.getAllKycApplications().subscribe({
      next: (data) => (this.applications = data),
      error: (err) => console.error('Error loading applications:', err),
    });
  }

  updateStatus(id: number, status: string): void {
    this.kycService.updateKycStatus(id, status).subscribe({
      next: () => {
        alert(`KYC ${status}`);
        this.fetchApplications(); // Refresh after action
      },
      error: (err) => console.error('Update failed:', err),
    });
  }
}
import { Routes } from '@angular/router';
import { CustomerRegisterComponent } from './customer-register/customer-register.component';
import { KycFormComponent } from './kyc-form/kyc-form.component';
import { KycStatusComponent } from './kyc-status/kyc-status.component';
import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';

export const routes: Routes = [
  { path: '', redirectTo: 'register', pathMatch: 'full' },
  { path: 'register', component: CustomerRegisterComponent },
  { path: 'kyc-form', component: KycFormComponent },
  { path: 'kyc-status', component: KycStatusComponent },
  { path: 'admin', component: AdminDashboardComponent },
];
<nav class="navbar navbar-expand-lg navbar-dark bg-dark px-3">
  <a class="navbar-brand" routerLink="/">KYC Portal</a>
  <div class="navbar-nav">
    <a class="nav-link" routerLink="/register">Register</a>
    <a class="nav-link" routerLink="/kyc-form">Submit KYC</a>
    <a class="nav-link" routerLink="/kyc-status">KYC Status</a>
    <a class="nav-link" routerLink="/admin">Admin Dashboard</a>
  </div>
</nav>

<div class="container mt-4">
  <router-outlet></router-outlet>
</div>
import { Component } from '@angular/core';
import { RouterModule } from '@angular/router';
import { routes } from './app.routes';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterModule.forRoot(routes)],
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent {}
