@Test
void testSubmitKycAfterRegisteringCustomer() throws Exception {
    // Step 1: Register customer
    CustomerDto customerDto = new CustomerDto();
    customerDto.setName("John Doe");
    customerDto.setEmail("john.doe@example.com");
    customerDto.setAddress("123 Street");
    customerDto.setPhoneNumber("1234567890");
    customerDto.setDateOfBirth(LocalDate.of(1990, 1, 1));

    String customerResponse = mockMvc.perform(post("/api/customer/register")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(customerDto)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.customerId").exists())
            .andReturn()
            .getResponse()
            .getContentAsString();

    // Extract customerId from response
    Long customerId = JsonPath.read(customerResponse, "$.customerId");

    // Step 2: Submit KYC
    MockMultipartFile file = new MockMultipartFile(
            "file",
            "doc.pdf",
            MediaType.APPLICATION_PDF_VALUE,
            "Dummy File Content".getBytes()
    );

    mockMvc.perform(multipart("/api/customer/submit-kyc")
            .file(file)
            .param("customerId", customerId.toString()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.message").value("KYC Submitted Successfully"))
            .andExpect(jsonPath("$.kycApplication").exists());
}
