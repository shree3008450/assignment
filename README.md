file.upload-dir=C:/Users/ssrivast022725/Documents/KycBackend/uploads
try (InputStream inputStream = file.getInputStream()) {
    Files.copy(inputStream, dest.toPath(), StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) {
    throw new RuntimeException("Failed to store file", e);
}
