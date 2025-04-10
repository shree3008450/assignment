@PostMapping("/submit-kyc")
public ResponseEntity<String> submitkyc(@RequestParam Long customerId,
                                        @RequestParam MultipartFile file) {
    kycService.submitKyc(file, customerId);
    return ResponseEntity.ok("KYC submitted successfully.");
}
