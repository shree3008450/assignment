@Override
public KycResponseDto submitkyc(MultipartFile file, Long customerId) {
    Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));

    KycApplication existing = kycRepository.findByCustomerId(customerId).orElse(null);

    if (existing != null && existing.getStatus() != KycStatus.REJECTED) {
        throw new IllegalStateException("You can't modify KYC unless it's rejected");
    }

    String uploadDir = env.getProperty("file.upload-dir");
    String filename = UUID.randomUUID() + "_" + file.getOriginalFilename();
    File uploadFolder = new File(uploadDir);

    if (!uploadFolder.exists()) {
        uploadFolder.mkdirs();
    }

    File dest = new File(uploadDir, filename);

    try (InputStream inputStream = file.getInputStream()) {
        Files.copy(inputStream, dest.toPath(), StandardCopyOption.REPLACE_EXISTING);
    } catch (IOException e) {
        throw new RuntimeException("Failed to store file", e);
    }

    KycApplication kyc;
    if (existing != null && existing.getStatus() == KycStatus.REJECTED) {
        kyc = existing;
    } else {
        kyc = new KycApplication();
        kyc.setCustomer(customer);
    }

    kyc.setDocumentPath(dest.getPath());
    kyc.setStatus(KycStatus.PENDING);
    kyc.setSubmittedAt(LocalDateTime.now());

    customer.setKycApplication(kyc);

    kycRepository.save(kyc);

    // Prepare DTOs for response
    CustomerDto customerDto = new CustomerDto();
    customerDto.setId(customer.getId());
    customerDto.setName(customer.getName());
    customerDto.setEmail(customer.getEmail());
    customerDto.setAddress(customer.getAddress());
    customerDto.setPhoneNumber(customer.getPhoneNumber());
    customerDto.setDateOfBirth(customer.getDateOfBirth());

    KycResponseDto response = new KycResponseDto();
    response.setId(kyc.getId());
    response.setDocumentPath(kyc.getDocumentPath());
    response.setStatus(kyc.getStatus().toString());
    response.setSubmittedAt(kyc.getSubmittedAt());
    response.setCustomer(customerDto);
    response.setMessage("KYC submitted successfully");

    return response;
}
