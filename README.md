@Override
    public void submitKyc(KycRequestDto dto) {
        Customer customer = customerRepository.findById(dto.getCustomerId())
                .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));

        KycApplication existing = kycRepository.findByCustomerId(customer.getId()).orElse(null);

        if (existing != null && existing.getStatus() != KycStatus.REJECTED) {
            throw new IllegalStateException("You can't modify KYC unless it's rejected");
        }

        KycApplication kyc = new KycApplication();
        kyc.setCustomer(customer);
        kyc.setDocumentPath(dto.getDocumentPath());
        kyc.setStatus(KycStatus.PENDING);
        kyc.setSubmittedAt(LocalDateTime.now());

        kycRepository.save(kyc);
    }
