package com.sgcib.fdc.search.infrastructure.remote.gilt;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonElement;
import com.google.gson.JsonParser;
import feign.Response;
import feign.codec.Decoder;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.io.IOUtils;
import org.springframework.cloud.openfeign.support.ResponseEntityDecoder;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.lang.reflect.Type;
import java.nio.charset.StandardCharsets;

@Slf4j
@Component
public class GiltDecodeConfig {

    @Bean
    public ResponseEntityDecoder giltResponseDecoder() {
        final var gson = new GsonBuilder().setLenient(true).create();

        return new ResponseEntityDecoder((Response response, Type type) -> {
            try {
                var stringResponse = IOUtils.toString(response.body().asInputStream(), StandardCharsets.UTF_8);
                
                // Handle empty responses
                if (stringResponse == null || stringResponse.trim().isEmpty()) {
                    return null;
                }
                
                // Create lenient JsonParser
                JsonElement json;
                try {
                    json = JsonParser.parseString(stringResponse);
                } catch (Exception ex) {
                    log.error("Error while parsing gilt response", ex);
                    log.error("Gilt error is: " + stringResponse);
                    return null;
                }
                
                if (json.isJsonArray()) {
                    return gson.fromJson(json, type);
                } else if (json.isJsonObject()) {
                    return gson.fromJson(json, type);
                } else {
                    // Handle error case
                    log.error("Unexpected JSON format: " + stringResponse);
                    return null;
                }
            } catch (Exception e) {
                log.error("Error decoding gilt response", e);
                return null;
            }
        });
    }
}
