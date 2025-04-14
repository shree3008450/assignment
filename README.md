@Test
void testGetAllApplications() throws Exception {
    // Arrange: Mock data
    KycResponseDto dto = new KycResponseDto();
    dto.setId(1L);
    dto.setStatus("APPROVED");
    dto.setDocumentPath("uploads/kyc.pdf");
    dto.setSubmittedAt(LocalDate.now());
    dto.setCustomerName("John Doe");

    List<KycResponseDto> kycList = List.of(dto);

    when(kycService.getAllApplications()).thenReturn(kycList);

    // Act & Assert
    mockMvc.perform(get("/api/admin/all-applications"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.size()").value(1))
            .andExpect(jsonPath("$[0].id").value(1))
            .andExpect(jsonPath("$[0].status").value("APPROVED"))
            .andExpect(jsonPath("$[0].documentPath").value("uploads/kyc.pdf"))
            .andExpect(jsonPath("$[0].customerName").value("John Doe"));
}
