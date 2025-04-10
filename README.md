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


