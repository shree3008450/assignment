@PostMapping("/register")
public ResponseEntity<Map<String, String>> register(@Valid @RequestBody CustomerDto dto) {
    customerService.registerCustomer(dto);
    Map<String, String> response = new HashMap<>();
    response.put("message", "Customer registered");
    return ResponseEntity.ok(response);
}
