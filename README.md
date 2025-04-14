@Test
void testFullKycFlow() throws Exception {
    // 1. Register customer
    String customerJson = """
        {
            "name": "John Doe",
            "email": "john@example.com",
            "address": "123 Main St",
            "phoneNumber": "1234567890",
            "dateOfBirth": "1990-01-01"
        }
    """;

    MvcResult registerResult = mockMvc.perform(post("/api/customer/register")
            .contentType(MediaType.APPLICATION_JSON)
            .content(customerJson))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.customerId").exists())
        .andReturn();

    String responseBody = registerResult.getResponse().getContentAsString();
    ObjectMapper mapper = new ObjectMapper();
    Long customerId = mapper.readTree(responseBody).get("customerId").asLong();

    // 2. Submit KYC
    MockMultipartFile document = new MockMultipartFile(
            "document", "kyc.pdf", MediaType.APPLICATION_PDF_VALUE, "Dummy KYC Document".getBytes());

    mockMvc.perform(multipart("/api/kyc/submit/{customerId}", customerId)
            .file(document)
            .param("submittedDate", "2025-04-13"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.message").value("KYC submitted successfully."));

    // 3. Get KYC Status
    mockMvc.perform(get("/api/customer/kyc-status/{id}", customerId))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.status").exists());

    // 4. Approve KYC
    mockMvc.perform(put("/api/admin/approve/{id}", customerId)
            .param("status", "APPROVED"))
        .andExpect(status().isOk())
        .andExpect(content().string("KYC status updated successfully"));

    // 5. Get all applications
    mockMvc.perform(get("/api/admin/all-applications"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$").isArray())
        .andExpect(jsonPath("$[0].customer.id").value(customerId));
}
