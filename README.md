<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
@Test
void testRegisterCustomer() throws Exception {
    CustomerDto customerDto = new CustomerDto("John", "john@mail.com", "123 St", "9876543210", LocalDate.of(1995, 5, 12));

    mockMvc.perform(post("/api/customers")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(customerDto)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").exists());
}
@Test
void testSubmitKyc() throws Exception {
    MockMultipartFile file = new MockMultipartFile("file", "document.pdf", "application/pdf", "dummy content".getBytes());

    mockMvc.perform(multipart("/api/customers/kyc")
            .file(file)
            .param("customerId", "1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.message").value("KYC submitted successfully"));
}
@Test
void testGetKycStatus() throws Exception {
    mockMvc.perform(get("/api/customers/kyc-status/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.kycApplication").exists());
}
@Test
void testGetPendingKycs() throws Exception {
    mockMvc.perform(get("/api/admin/pending-kycs"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$").isArray());
}
@Test
void testApproveKyc() throws Exception {
    mockMvc.perform(put("/api/admin/kyc/1")
            .param("status", "ACCEPTED"))
            .andExpect(status().isOk())
            .andExpect(content().string("KYC status updated successfully"));
}
@Test
void testRejectKyc() throws Exception {
    mockMvc.perform(put("/api/admin/kyc/1")
            .param("status", "REJECTED"))
            .andExpect(status().isOk())
            .andExpect(content().string("KYC status updated successfully"));
}

