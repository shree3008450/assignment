GiltAdapter spyAdapter = spy(giltAdapter);
    
    // Override the getGiltByCode method to use giltListId instead of code
    doAnswer(invocation -> {
        // Ignore the code parameter and use giltListId instead
        ResponseEntity<Gilt> response = giltApiClient.fetchGilt(null, giltListId);
        return response != null ? response.getBody() : null;
    }).when(spyAdapter).getGiltByCode(anyString());
    
    // When - first call
    spyAdapter.getGiltByCode(giltCode);
    
    // When - second call (should use cache)
    spyAdapter.getGiltByCode(giltCode);
    
