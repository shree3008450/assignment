import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-kyc-form',
  standalone: true,
  templateUrl: './kyc-form.component.html',
  styleUrls: ['./kyc-form.component.css']
})
export class KycFormComponent {
  selectedFile: File | null = null;
  customerId: number = 0;

  constructor(private http: HttpClient) {}

  onFileSelected(event: any) {
    this.selectedFile = event.target.files[0];
  }

  submitKyc() {
    if (!this.selectedFile || !this.customerId) return;

    const formData = new FormData();
    formData.append('file', this.selectedFile);
    formData.append('customerId', this.customerId.toString());

    this.http.post('http://localhost:8080/api/customer/submit-kyc', formData)
      .subscribe(response => {
        console.log('KYC submitted', response);
      });
  }
}
<div class="container mt-4">
  <h3>Submit KYC</h3>
  <div class="form-group">
    <label>Customer ID:</label>
    <input type="number" [(ngModel)]="customerId" class="form-control" />
  </div>

  <div class="form-group mt-2">
    <label>Upload Document:</label>
    <input type="file" (change)="onFileSelected($event)" class="form-control" />
  </div>

  <button (click)="submitKyc()" class="btn btn-primary mt-3">Submit KYC</button>
</div>
<div class="container mt-4">
  <h3>Register Customer</h3>

  <form (ngSubmit)="registerCustomer()">
    <div class="form-group">
      <label>Name</label>
      <input class="form-control" [(ngModel)]="customer.name" name="name" required />
    </div>

    <div class="form-group mt-2">
      <label>Email</label>
      <input class="form-control" [(ngModel)]="customer.email" name="email" required type="email" />
    </div>

    <div class="form-group mt-2">
      <label>Address</label>
      <input class="form-control" [(ngModel)]="customer.address" name="address" required />
    </div>

    <div class="form-group mt-2">
      <label>Phone</label>
      <input class="form-control" [(ngModel)]="customer.phoneNumber" name="phoneNumber" required />
    </div>

    <div class="form-group mt-2">
      <label>Date of Birth</label>
      <input type="date" class="form-control" [(ngModel)]="customer.dateOfBirth" name="dateOfBirth" required />
    </div>

    <button type="submit" class="btn btn-success mt-3">Register</button>
  </form>
</div>
import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-customer-register',
  standalone: true,
  templateUrl: './customer-register.component.html',
  styleUrls: ['./customer-register.component.css']
})
export class CustomerRegisterComponent {
  customer = {
    name: '',
    email: '',
    address: '',
    phoneNumber: '',
    dateOfBirth: ''
  };

  constructor(private http: HttpClient) {}

  registerCustomer() {
    this.http.post('http://localhost:8080/api/customer/register', this.customer)
      .subscribe(response => {
        console.log('Customer registered', response);
      });
  }
}
