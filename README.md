import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class KycService {

  private baseUrl = 'http://localhost:8080/api'; // update if needed

  constructor(private http: HttpClient) {}

  registerCustomer(customerData: any): Observable<any> {
    return this.http.post(`${this.baseUrl}/customers`, customerData);
  }

  submitKyc(formData: FormData): Observable<any> {
    return this.http.post(`${this.baseUrl}/kyc`, formData);
  }

  getKycStatus(customerId: number): Observable<any> {
    return this.http.get(`${this.baseUrl}/kyc/status/${customerId}`);
  }

  getAllKycApplications(): Observable<any[]> {
    return this.http.get<any[]>(`${this.baseUrl}/admin/kyc`);
  }

  updateKycStatus(id: number, status: string): Observable<any> {
    return this.http.put(`${this.baseUrl}/admin/kyc/${id}?status=${status}`, {});
  }
}
