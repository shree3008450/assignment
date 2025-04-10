this.kycService.registerCustomer(this.customerForm.value).subscribe({
  next: (res) => {
    console.log('Registered:', res);
    const customerId = res.customerId;
    localStorage.setItem('customerId', customerId.toString()); // Save for later use
    alert('Registered successfully! Customer ID: ' + customerId);
  },
  error: (err) => {
    console.error('Error registering:', err);
    alert('Registration failed');
  }
});
