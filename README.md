import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-kyc-status',
  standalone: true,
  templateUrl: './kyc-status.component.html',
  styleUrls: ['./kyc-status.component.css']
})
export class KycStatusComponent {
  email = '';
  status: string | null = null;
  error: string | null = null;

  constructor(private http: HttpClient) {}

  checkStatus() {
    this.http.get<any>(`http://localhost:8080/api/customer/status?email=${this.email}`)
      .subscribe({
        next: (response) => {
          this.status = response.status;
          this.error = null;
        },
        error: () => {
          this.status = null;
          this.error = 'No KYC found for this email';
        }
      });
  }
}
<div class="container mt-4">
  <h3>Check KYC Status</h3>

  <form (ngSubmit)="checkStatus()">
    <div class="form-group">
      <label>Email:</label>
      <input type="email" [(ngModel)]="email" name="email" class="form-control" required>
    </div>

    <button type="submit" class="btn btn-primary mt-2">Check Status</button>
  </form>

  <div *ngIf="status" class="alert alert-info mt-3">
    KYC Status: <strong>{{ status }}</strong>
  </div>

  <div *ngIf="error" class="alert alert-danger mt-3">
    {{ error }}
  </div>
</div>
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-admin-dashboard',
  standalone: true,
  templateUrl: './admin-dashboard.component.html',
  styleUrls: ['./admin-dashboard.component.css']
})
export class AdminDashboardComponent implements OnInit {
  applications: any[] = [];

  constructor(private http: HttpClient) {}

  ngOnInit(): void {
    this.fetchPendingApplications();
  }

  fetchPendingApplications() {
    this.http.get<any[]>('http://localhost:8080/api/admin/pending-kycs')
      .subscribe(data => this.applications = data);
  }

  updateStatus(id: number, status: string) {
    this.http.put(`http://localhost:8080/api/admin/update-status/${id}?status=${status}`, null)
      .subscribe(() => {
        this.fetchPendingApplications(); // refresh list
      });
  }
}
<div class="container mt-4">
  <h3>Pending KYC Applications</h3>

  <table class="table table-bordered mt-3" *ngIf="applications.length > 0">
    <thead class="table-dark">
      <tr>
        <th>ID</th>
        <th>Customer Name</th>
        <th>Document</th>
        <th>Submitted At</th>
        <th>Action</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let app of applications">
        <td>{{ app.id }}</td>
        <td>{{ app.customer.name }}</td>
        <td>
          <a [href]="'http://localhost:8080/uploads/' + app.documentPath" target="_blank">View Document</a>
        </td>
        <td>{{ app.submittedAt | date:'short' }}</td>
        <td>
          <button class="btn btn-success btn-sm me-1" (click)="updateStatus(app.id, 'APPROVED')">Approve</button>
          <button class="btn btn-danger btn-sm" (click)="updateStatus(app.id, 'REJECTED')">Reject</button>
        </td>
      </tr>
    </tbody>
  </table>

  <div *ngIf="applications.length === 0" class="alert alert-info">
    No pending applications.
  </div>
</div>
