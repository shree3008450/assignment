package com.kyc.controller;

import com.kyc.dto.CustomerDto;
import com.kyc.dto.KycRequestDto;
import com.kyc.dto.KycResponseDto;
import com.kyc.service.CustomerService;
import com.kyc.service.KycService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/customer")
public class CustomerController {

    @Autowired
    private CustomerService customerService;

    @Autowired
    private KycService kycService;

    @PostMapping("/register")
    public String register(@RequestBody CustomerDto dto) {
        customerService.registerCustomer(dto);
        return "Customer registered";
    }

    @PostMapping("/submit-kyc")
    public String submitKyc(@RequestBody KycRequestDto dto) {
        kycService.submitKyc(dto);
        return "KYC submitted";
    }

    @GetMapping("/kyc-status/{customerId}")
    public KycResponseDto getKycStatus(@PathVariable Long customerId) {
        return customerService.getKycStatus(customerId);
    }
}
