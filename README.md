import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-kyc-form',
  standalone: true,
  templateUrl: './kyc-form.component.html',
  styleUrls: ['./kyc-form.component.css'],
})
export class KycFormComponent implements OnInit {
  selectedFile!: File;
  customerId!: number;

  constructor(private http: HttpClient) {}

  ngOnInit(): void {
    const storedId = localStorage.getItem('customerId');
    if (storedId) {
      this.customerId = +storedId;
    } else {
      console.error('No customerId found in localStorage');
    }
  }

  onFileSelected(event: any): void {
    this.selectedFile = event.target.files[0];
  }

  submitKyc(): void {
    if (!this.selectedFile || !this.customerId) {
      console.warn('File or Customer ID missing');
      return;
    }

    const formData = new FormData();
    formData.append('file', this.selectedFile);
    formData.append('customerId', this.customerId.toString());

    this.http.post('http://localhost:8080/api/customer/submit-kyc', formData)
      .subscribe({
        next: (response) => {
          console.log('KYC submitted', response);
        },
        error: (error) => {
          console.error('Submission failed', error);
        }
      });
  }
}
<div class="container mt-3">
  <h3>Submit KYC</h3>

  <div class="mb-3">
    <label for="file" class="form-label">Upload Document</label>
    <input type="file" id="file" (change)="onFileSelected($event)" class="form-control">
  </div>

  <button (click)="submitKyc()" class="btn btn-primary">Submit</button>
</div>
