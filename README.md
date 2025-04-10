public void submitKyc(MultipartFile file, Long customerId) {
    Customer customer = customerRepository.findById(customerId)
        .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));

    KycApplication existing = kycRepository.findByCustomerId(customer.getId()).orElse(null);
    if (existing != null && existing.getStatus() != KycStatus.REJECTED) {
        throw new IllegalStateException("You can't modify KYC unless it's rejected");
    }

    // Save file to uploads folder
    String uploadDir = env.getProperty("file.upload-dir");
    String filename = UUID.randomUUID() + "_" + file.getOriginalFilename();
    File dest = new File(uploadDir, filename);

    try {
        file.transferTo(dest);
    } catch (IOException e) {
        throw new RuntimeException("Failed to store file", e);
    }

    KycApplication kyc = new KycApplication();
    kyc.setCustomer(customer);
    kyc.setDocumentPath(dest.getPath());  // saved path
    kyc.setStatus(KycStatus.PENDING);
    kyc.setSubmittedAt(LocalDateTime.now());

    kycRepository.save(kyc);
}
