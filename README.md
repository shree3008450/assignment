@Override
public KycResponseDto submitkyc(MultipartFile file, Long customerId) {
    Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));

    Optional<KycApplication> existingOpt = kycRepository.findByCustomerId(customer.getId());
    KycApplication kyc;

    if (existingOpt.isPresent()) {
        kyc = existingOpt.get();

        if (kyc.getStatus() != KycStatus.REJECTED) {
            throw new IllegalStateException("You can't modify KYC unless it's rejected.");
        }
        // Overwrite the same KYC entry
    } else {
        kyc = new KycApplication();
        kyc.setCustomer(customer);
    }

    // File storage logic
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

    // Update KYC application details
    kyc.setDocumentPath(dest.getPath());
    kyc.setStatus(KycStatus.PENDING);
    kyc.setSubmittedAt(LocalDateTime.now());

    // Maintain bidirectional mapping
    customer.setKycApplication(kyc);

    kycRepository.save(kyc);

    // Create and return response DTO
    CustomerDto customerDto = new CustomerDto(customer);

    return new KycResponseDto(
            "KYC submitted successfully",
            kyc.getId(),
            kyc.getDocumentPath(),
            kyc.getStatus(),
            kyc.getSubmittedAt(),
            customerDto
    );
}
