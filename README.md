import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class KycService {

  private baseUrl = 'http://localhost:8080/api/customer'; // ✅ match your backend @RequestMapping

  constructor(private http: HttpClient) {}

  // ✅ Customer Registration
  registerCustomer(customerData: any): Observable<any> {
    return this.http.post(`${this.baseUrl}/register`, customerData);
  }

  // ✅ Submit KYC (with file upload)
  submitKyc(formData: FormData): Observable<any> {
    return this.http.post(`${this.baseUrl}/submit-kyc`, formData);
  }

  // ✅ Get KYC status for a customer
  getKycStatus(customerId: number): Observable<any> {
    return this.http.get(`${this.baseUrl}/kyc-status/${customerId}`);
  }

  // ✅ Admin: Fetch all pending KYC applications
  getAllKycApplications(): Observable<any[]> {
    return this.http.get<any[]>(`${this.baseUrl}/pending`);
  }

  // ✅ Admin: Approve or Reject KYC by application ID
  updateKycStatus(id: number, status: string): Observable<any> {
    return this.http.post(`${this.baseUrl}/${status}/${id}`, {}); // POST to /approve/{id} or /reject/{id}
  }
}
