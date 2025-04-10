@PostMapping(value = "/submit-kyc", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public String submitKyc(@RequestParam("customerId") Long customerId,
                        @RequestParam("file") MultipartFile file) {
    kycService.submitKyc(file, customerId);
    return "KYC submitted";
}
