 submitKyc(file: File, customerId: number): Observable<any> {
    const formData = new FormData();
    formData.append('file', file);
    formData.append('customerId', customerId.toString());

    return this.http.post(`${this.baseUrl}/submit`, formData);
  }
  import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { KycService } from '../services/kyc.service';

@Component({
  selector: 'app-kyc-form',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './kyc-form.component.html',
  styleUrls: ['./kyc-form.component.css']
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

    this.kycService.submitKyc(this.selectedFile, +customerId).subscribe({
      next: () => alert('KYC submitted!'),
      error: (err) => {
        console.error('Error submitting KYC:', err);
        alert('Submission failed');
      }
    });
  }
}
